<template>
  <div>
    <v-container fluid grid-list-md>
      <v-card>
        <v-toolbar dense>
          <v-toolbar-title>呪文検索</v-toolbar-title>
          <v-spacer></v-spacer>
          <file-upload-icon
            tooltip="呪文データファイルをアップロードします。"
            @onFileRead="onFileRead"
          />
          <v-tooltip top>
            <v-btn icon small slot="activator" @click="saveSpellData">
              <v-icon>save</v-icon>
            </v-btn>
            <span>ブラウザのローカルストレージに呪文データを保存します。</span>
          </v-tooltip>
          <file-download-icon
            tooltip="現在の呪文データを、ファイルにダウンロードします。"
            :data="jesonspelldata"
          />
          <v-tooltip top>
            <v-btn icon small slot="activator" @click="loadSpellData">
              <v-icon>redo</v-icon>
            </v-btn>
            <span
              >ブラウザのローカルストレージに保存されている呪文データを再ロードします。</span
            >
          </v-tooltip>
          <v-tooltip top>
            <v-btn icon small slot="activator" @click="deleteSpellData">
              <v-icon>delete</v-icon>
            </v-btn>
            <span
              >ブラウザのローカルストレージから呪文データを全削除します。</span
            >
          </v-tooltip>
          <v-tooltip top>
            <v-btn icon small slot="activator" @click="addSpell">
              <v-icon>add</v-icon>
            </v-btn>
            <span>呪文を追加します。</span>
          </v-tooltip>
          <v-tooltip top>
            <v-btn icon small slot="activator" @click="showHelp">
              <v-icon>help</v-icon>
            </v-btn>
            <span>ヘルプを表示します</span>
          </v-tooltip>
          <v-tooltip top>
            <v-btn
              icon
              small
              slot="activator"
              @click="() => (this.microphone = !this.microphone)"
            >
              <v-icon
                >{{
                  this.microphone
                    ? "fas fa-microphone"
                    : "fas fa-microphone-slash"
                }}
              </v-icon>
            </v-btn>
            <span>音声入力の切り替えをします(Google Chromeのみ)</span>
          </v-tooltip>
        </v-toolbar>

        <!--デスクトップ用-->
        <!--検索条件-->
        <v-layout v-if="!isMobile" row wrap justify-center>
          <v-flex xs1> </v-flex>
          <v-flex xs3>
            <v-text-field
              append-icon="search"
              row-height="12"
              label="Input"
              single-line
              v-model="conditon.spellname"
              hint="呪文名"
              persistent-hint
            ></v-text-field>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="levels"
              v-model="conditon.levels"
              multiple
              max-height="400"
              hint="呪文レベル"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="casting_time"
              v-model="conditon.casting_time"
              max-height="400"
              hint="詠唱時間"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs2>
            <v-select
              dense
              label="Select"
              :items="classes"
              v-model="conditon.classes"
              multiple
              max-height="400"
              hint="取得クラス"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs2>
            <v-select
              dense
              label="Select"
              :items="classes"
              v-model="conditon.exlcudedClasses"
              multiple
              max-height="400"
              hint="除外クラス(検索対象の呪文を取得していないクラスを指定します"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="sources"
              v-model="conditon.sources"
              multiple
              max-height="400"
              hint="出典"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1> </v-flex>
          <v-flex xs1> </v-flex>
          <v-flex xs3>
            <v-text-field
              dense
              append-icon="search"
              row-height="12"
              label="Input"
              single-line
              v-model="conditon.desc"
              hint="本文（スペース区切りで複数条件指定可能。単語の直前に|を入力するとOR条件で評価されます。）"
              persistent-hint
            ></v-text-field>
          </v-flex>
          <v-flex xs2>
            <v-select
              dense
              label="Select"
              :items="subclassses"
              v-model="conditon.subclassses"
              multiple
              max-height="400"
              hint="サブクラス"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="schools"
              v-model="conditon.school"
              max-height="400"
              hint="系統"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="concentration"
              v-model="conditon.concentration"
              max-height="400"
              hint="集中"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              :items="conditonRituals"
              v-model="conditon.ritual"
              max-height="400"
              hint="儀式発動"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="components"
              v-model="conditon.components"
              multiple
              max-height="400"
              hint="構成要素"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1>
            <v-select
              dense
              label="Select"
              :items="allTags"
              v-model="conditon.tags"
              item-text="name"
              item-value="self"
              multiple
              max-height="400"
              hint="タグ"
              persistent-hint
            ></v-select>
          </v-flex>
          <v-flex xs1> </v-flex>
        </v-layout>
        <!--検索結果-->
        <v-data-table
          v-if="!isMobile"
          :headers="headers"
          :items="items"
          item-key="name"
          no-data-text="条件に一致する呪文がありません。"
        >
          <template slot="items" slot-scope="props">
            <tr
              @click="clickCell(props.item)"
              @click.right.prevent="rightClickCell(props.item)"
            >
              <td
                class="
              text-xs-left"
              >
                {{ formatSpellName(props.item) }}
              </td>
              <td class="text-xs-left">
                {{ props.item.hoge }} {{ props.item.level }}
              </td>
              <td class="text-xs-left">
                {{ props.item.formatArray(props.item.components, components) }}
              </td>
              <td class="text-xs-left nowrap">{{ props.item.casting_time }}</td>
              <td class="text-xs-left">{{ props.item.formatDuration }}</td>
              <td class="text-xs-left">{{ props.item.range }}</td>
              <td class="text-xs-left">{{ props.item.source }}</td>
              <td class="text-xs-left">
                <v-icon>add</v-icon>
              </td>
            </tr>
          </template>
        </v-data-table>
        <!--デスクトップ用ここまで-->

        <!--モバイル-->
        <!--検索条件-->
        <v-layout v-if="isMobile" row wrap>
          <v-flex xs10 offset-xs1>
            <v-text-field
              append-icon="search"
              label="Input"
              single-line
              v-model="conditon.spellname"
              hint="呪文名"
              persistent-hint
            ></v-text-field>
          </v-flex>
          <v-flex xs6 offset-xs3>
            <v-btn @click="showMobileSearchDetailDialog = true">
              <v-icon>settings</v-icon> 詳細検索条件
            </v-btn>
          </v-flex>
        </v-layout>
        <!--検索結果-->
        <v-list v-if="isMobile" dense two-line>
          <v-data-iterator content-tag="v-card" :items="items">
            <v-list-tile
              avatar
              slot="item"
              slot-scope="props"
              @click="clickCell(props.item)"
            >
              <v-list-tile-content>
                <v-list-tile-title>{{ props.item.name }} </v-list-tile-title>
                <v-list-tile-sub-title
                  >{{ props.item.level }}/{{
                    props.item.format(props.item.school, schools)
                  }}/{{
                    props.item.formatArray(props.item.components, components)
                  }}</v-list-tile-sub-title
                >
              </v-list-tile-content>
              <v-list-tile-avatar>
                <v-icon>add</v-icon>
              </v-list-tile-avatar>
            </v-list-tile>
          </v-data-iterator>
        </v-list>

        <!--モバイル　ここまで-->

        <v-card-text
          >2018.09.02
          サブクラスで呪文を絞り込む機能を追加しました。呪文データの再取得と再ロードをお願いします。手順がわからない人は、minokubaまで連絡下さい。</v-card-text
        >
      </v-card>
      <v-card>
        <ins
          class="adsbygoogle"
          style="display:block"
          data-ad-client="ca-pub-9097509632200457"
          data-ad-slot="5985300299"
          data-ad-format="auto"
        ></ins>
      </v-card>
    </v-container>
    <v-snackbar
      top
      :color="snackbar.level"
      :timeout="3000"
      v-model="snackbar.show"
    >
      {{ snackbar.message }}
    </v-snackbar>
    <spell-detail-dialog
      :showEditDialog.sync="showEditDialog"
      v-bind:targetSpell="targetSpell"
      v-bind:createSpell="createSpell"
      :tags="allTags"
      @save="save"
      @remove="remove"
      @cancel="cancelOrClose"
      @close="cancelOrClose"
      @editTag="editTagFromDetailDalog"
    />
    <spell-help-dialog :showDialog.sync="showHelpDialog"></spell-help-dialog>
    <add-tag-dialog
      :showDialog.sync="showAddTagDialog"
      :tags.sync="allTags"
      :spellname="addTagSpellName"
      @save="saveTags"
      @cancel="cancelAddTags"
    />

    <v-dialog v-if="isMobile" fullscreen v-model="showMobileSearchDetailDialog">
      <v-card>
        <v-card-title>
          <H2>検索詳細条件入力</H2>
        </v-card-title>
        <v-card>
          <v-layout row wrap>
            <v-flex xs10 offset-xs1>
              <v-select
                label="Select"
                hide-selected
                dense
                :items="levels"
                v-model="conditon.levels"
                multiple
                max-height="200"
                hint="呪文レベル"
                persistent-hint
              ></v-select>
            </v-flex>
            <v-flex xs10 offset-xs1>
              <v-select
                label="Select"
                hide-selected
                dense
                :items="classes"
                v-model="conditon.classes"
                multiple
                max-height="200"
                hint="クラス"
                persistent-hint
              ></v-select>
            </v-flex>
            <!--v-flex xs10 offset-xs1>
              <v-select label="Select" hide-selected dense :items="subclassses" v-model="conditon.subclassses" multiple max-height="200" hint="サブクラス" persistent-hint></v-select>
            </v-flex-->
            <v-flex xs10 offset-xs1>
              <v-select
                :items="conditonRituals"
                dense
                v-model="conditon.ritual"
                max-height="200"
                hint="儀式発動"
                persistent-hint
              ></v-select>
            </v-flex>
            <v-flex xs10 offset-xs1>
              <v-select
                label="Select"
                dense
                :items="components"
                v-model="conditon.components"
                multiple
                max-height="200"
                hint="構成要素"
                persistent-hint
              ></v-select>
            </v-flex>
            <v-flex xs10 offset-xs1>
              <v-select
                label="Select"
                dense
                :items="casting_time"
                v-model="conditon.casting_time"
                max-height="200"
                hint="詠唱時間"
                persistent-hint
              ></v-select>
            </v-flex>
            <!--v-flex xs10 offset-xs1>
              <v-select label="Select" dense :items="schools" v-model="conditon.school" max-height="200" hint="系統" persistent-hint></v-select>
            </v-flex-->
          </v-layout>
        </v-card>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="green darken-1"
            flat
            @click.native="closeMobileSearchDetailDialog"
            >閉じる</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<style>
/*table-layout: fixed; せずに text-overflow: ellipsis; する方法（長すぎる文字を...で省略表示する。
 * http://chuckwebtips.hatenablog.com/entry/2017/07/09/183527 
 */
td.nowrap {
  max-width: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  -o-text-overflow: ellipsis;
}
</style>

<script>
import spells from "@/model/spells";
import constants from "@/model/constants";
import Spell from "@/model/spell";
import Tag from "@/model/tag";

import FileUploadIcon from "@/components/FileUploadIcon";
import FileDownloadIcon from "@/components/FileDownloadIcon";
import SpellDetailDialog from "@/components/SpellDetailDialog";
import SpellHelpDialog from "@/components/SpellHelpDialog";
import AddTagDialog from "@/components/AddTagDialog";

export default {
  name: "SearchSpellTable",
  components: {
    FileUploadIcon: FileUploadIcon,
    FileDownloadIcon: FileDownloadIcon,
    SpellDetailDialog: SpellDetailDialog,
    SpellHelpDialog: SpellHelpDialog,
    AddTagDialog: AddTagDialog
  },
  mounted() {
    (window.adsbygoogle = window.adsbygoogle || []).push({});
  },
  beforeDestroy() {
    //web speech apiを解放
    this.microphone = false;
  },
  data() {
    return {
      //管理している呪文データ。
      spelldata: [],
      //検索条件
      conditon: {
        spellname: "",
        desc: "",
        levels: [],
        classes: [],
        exlcudedClasses: [],
        ritual: null,
        components: [],
        casting_time: "",
        concentration: null,
        school: null,
        subclassses: [],
        sources: [],
        tags: []
      },
      //アラートダイアログ
      snackbar: {
        show: false,
        level: "info",
        message: ""
      },
      //検索ダイアログに出すラベル管理。
      levels: constants.levels,
      classes: constants.classes,
      components: constants.components,
      concentration: constants.concentration,
      schools: constants.schools,
      casting_time: constants.casting_time,
      subclassses: constants.subclassses,
      sources: constants.sources,
      headers: [
        { text: "名前", value: "name", align: "left", width: "30%" },
        { text: "レベル", value: "level", align: "left", width: "5%" },
        { text: "構成要素", value: "components", align: "left", width: "20%" },
        {
          text: "詠唱時間",
          value: "casting_time",
          align: "left",
          width: "10%"
        },
        { text: "持続時間", value: "duration", align: "left", width: "10%" },
        { text: "距離／エリア", value: "range", align: "left", width: "10%" },
        { text: "出典", value: "source", align: "left", width: "10%" },
        { text: " ", value: "dummy", align: "left", width: "5%" }
      ],
      conditonRituals: [
        { text: "ー", value: null },
        { text: "可", value: "yes" },
        { text: "不可", value: "no" }
      ],
      allTags: [],
      //編集ダイアログ表示フラグ
      showEditDialog: false,
      //編集モードとするかの指定
      createSpell: false,
      //編集ダイアログに渡した呪文データのポインタ。
      targetSpell: null,
      //ヘルプダイアログ表示フラグ
      showHelpDialog: false,
      //呪文検索条件詳細入力ダイアログ
      showMobileSearchDetailDialog: false,
      //呪文タグダイアログ
      showAddTagDialog: false,
      //タグに追加する呪文名
      addTagSpellName: null,

      //音声認識
      microphone: false,
      recognition: null
    };
  },

  beforeMount() {
    //1.ローカルストレージをチェックする
    var spellsjson = localStorage.getItem("dndapp.spells");
    //2.ストレージに登録されていればそのデータを復元。
    if (spellsjson != undefined && spellsjson != null) {
      //初期表示用の呪文データを取得する。
      this.spelldata = Spell.assigns(JSON.parse(spellsjson));
    } else {
      //2.ストレージに登録されていなければデフォルトデータを読込の上、デフォルトデータをセーブ。
      this.spelldata = Spell.assigns(spells());
    }

    this.recognition = this.getSpeechRecognition();

    this.allTags = Tag.load();
  },
  computed: {
    //データテーブルに表示する呪文一覧を絞り込み返却する。
    items() {
      console.log(this.conditon.classes);
      return (
        this.spelldata
          // .slice(1, 10) //for debug.
          .filter(element => {
            if (!this.filterDesc(element)) {
              return false;
            }
            if (!this.filterSpellname(element)) {
              return false;
            }
            if (!this.fliterRitual(element)) {
              return false;
            }
            if (!this.fliterLevels(element)) {
              return false;
            }
            //クラスとサブクラスの検索は、どちらか一方にヒットしたら一覧に出すようにする。
            if (!this.filterClassOrSubclass(element)) {
              return false;
            }
            if (!this.fliterComopnents(element)) {
              return false;
            }
            if (!this.filterCastingTime(element)) {
              return false;
            }
            if (!this.filterConcentration(element)) {
              return false;
            }
            if (!this.filterSchool(element)) {
              return false;
            }
            if (!this.fliterSources(element)) {
              return false;
            }
            if (!this.filterTags(element)) {
              return false;
            }
            return true;
          })
          .sort((one, two) => {
            return one.name < two.name ? -1 : 1;
          })
      );
    },
    //ダウンロード時の文字列を返却する。
    jesonspelldata() {
      return JSON.stringify(this.spelldata);
    },
    //モバイル用画面で表示するかどうかを判定する。
    //(モバイル/デスクトップのブレークポイントの切り替え時に、不要なDOMを作成しないよう、v-ifでDOM作成自体をスキップする。
    // 呪文結果一覧や検索条件などで、不要なwatchの監視が重いのではないかと想像し、そのような対応を行う。)
    isMobile() {
      const mobileBreakpoints = ["xs", "sm"];
      return mobileBreakpoints.some(e => {
        return this.$vuetify.breakpoint.name === e;
      });
    }
  },

  watch: {
    //音声入力のマイクON/OFFの制御
    //TODO リファクタリングしたいがどうしようか、、
    microphone(value) {
      if (value && !this.recognition) {
        this.microphone = false;
      } else {
        if (value && this.recognition) {
          if (!this.recognition.onstart) {
            this.recognition.onstart = () => {
              // console.log("start");
            };
            this.recognition.onend = () => {
              // console.log("end");
              if (this.microphone) {
                this.recognition.start();
              }
            };
            this.recognition.onresult = event => {
              // console.log(`onresult ${event}`);
              if (event.results.length > 0) {
                let text = event.results[0][0].transcript;
                //commnad
                window.console.log(text);

                //dialogが開いていない時に受け付けるコマンド
                if (!this.showEditDialog) {
                  if (text === "クリア" || text === "クリアー") {
                    this.clerConditon();
                    this.showSnackbar("success", "検索条件をクリアしました。");
                  } else if (
                    text.startsWith("レベル") ||
                    text.startsWith("呪文 レベル")
                  ) {
                    let level = text.replace(/レベル|呪文 レベル|\s/g, "");
                    if (level.indexOf("クリア") >= 0) {
                      this.conditon.levels = [];
                      this.showSnackbar("success", "レベルをクリアしました。");
                    } else {
                      level = level.toLowerCase();
                      // console.log(level);
                      const results = level.match(
                        /[1-9]|cantrip|キャントリップ/g
                      );
                      if (results) {
                        const levels = [];
                        results.forEach(result => {
                          if (
                            result === "cantrip" ||
                            result === "キャントリップ"
                          ) {
                            levels.push("Cantrip");
                          } else if (result === "1") {
                            levels.push("1st-level");
                          } else if (result === "2") {
                            levels.push("2nd-level");
                          } else if (result === "3") {
                            levels.push("3rd-level");
                          } else {
                            levels.push(`${result}th-level`);
                          }
                        });
                        if (levels.length > 0) {
                          this.conditon.levels = levels;
                        }
                      }
                    }
                  } else if (
                    text.startsWith("呪文名") ||
                    text.startsWith("呪文") ||
                    text.startsWith("名前")
                  ) {
                    let command = text.replace(/呪文名|呪文|名前|\s/g, "");
                    if (command.indexOf("クリア") >= 0) {
                      this.conditon.spellname = "";
                      this.showSnackbar("success", "呪文名をクリアしました。");
                    } else {
                      command = command.replace(/\s/, "・");
                      window.console.log(command);
                      this.conditon.spellname = command;
                    }
                  } else if (text.startsWith("本文")) {
                    let command = text.replace(/本文|\s/g, "");
                    if (command.indexOf("クリア") >= 0) {
                      this.conditon.desc = "";
                      this.showSnackbar("success", "本文をクリアしました。");
                    } else {
                      window.console.log(command);
                      this.conditon.desc = command;
                    }
                  } else if (text.startsWith("タグ")) {
                    let command = text.replace(/タグ|\s/g, "");
                    if (command.indexOf("クリア") >= 0) {
                      this.conditon.tags = [];
                      this.showSnackbar("success", "タグをクリアしました。");
                    } else {
                      const tags = [];
                      this.allTags.forEach(tag => {
                        if (command.indexOf(tag.name) >= 0) {
                          tags.push(tag);
                        }
                      });
                      if (tags.length > 0) {
                        this.conditon.tags = tags;
                      } else {
                        this.showSnackbar(
                          "error",
                          `タグを選択できません。認識した文字列： ${command}`
                        );
                      }
                    }
                  } else if (text.startsWith("クラス")) {
                    let command = text.replace(/クラス|\s/g, "");
                    if (command.indexOf("クリア") >= 0) {
                      this.conditon.classes = [];
                      this.showSnackbar("success", "クラスをクリアしました。");
                    } else {
                      const classes = [];
                      constants.classes.forEach(clazz => {
                        if (command.indexOf(clazz.text) >= 0) {
                          classes.push(clazz.value);
                        }
                      });
                      if (classes.length > 0) {
                        this.conditon.classes = classes;
                      } else {
                        this.showSnackbar(
                          "error",
                          `クラスを選択できません。認識した文字列： ${command}`
                        );
                      }
                    }
                  } else if (text.startsWith("表示")) {
                    const spells = this.items;
                    if (spells.length > 0) {
                      this.clickCell(spells[0]);
                    }
                  } else {
                    text = text.replace(/\s/, "・");
                    window.console.log(text);
                    this.conditon.spellname = text;
                  }
                  //dilaog表示中の操作
                } else if (text.startsWith("次")) {
                  const index = this.items.indexOf(this.targetSpell);
                  const next = this.items[index + 1];
                  if (next) {
                    this.clickCell(next);
                  }
                } else if (text.startsWith("前")) {
                  const index = this.items.indexOf(this.targetSpell);
                  const next = this.items[index - 1];
                  if (next) {
                    this.clickCell(next);
                  }
                } else if (text.startsWith("閉")) {
                  this.showEditDialog = false;
                }
              }
            };
          }
          this.recognition.start();
        } else if (!value && this.recognition) {
          this.recognition.stop();
        }
      }
    }
  },

  methods: {
    clerConditon() {
      this.conditon = {
        spellname: "",
        desc: "",
        levels: [],
        classes: [],
        exlcudedClasses: [],
        ritual: null,
        components: [],
        casting_time: "",
        concentration: null,
        school: null,
        subclassses: [],
        sources: [],
        tags: []
      };
    },
    //検索条件に従い、フィルタを行うメソッド。computedの中で呼び出されるのみで、テンプレートから直接呼ばれることはない。
    fliterRitual(element) {
      if (this.conditon.ritual === null) {
        return true;
      } else {
        return element.ritual === this.conditon.ritual;
      }
    },
    fliterLevels(element) {
      if (this.conditon.levels.length === 0) {
        return true;
      } else {
        return this.conditon.levels.some(e => {
          return e === element.level;
        });
      }
    },
    fliterSources(element) {
      if (this.conditon.sources.length === 0) {
        return true;
      } else {
        return this.conditon.sources.some(e => {
          return e === element.source;
        });
      }
    },

    filterClassOrSubclass(element) {
      //1.クラス・サブクラスの指定がない場合は、クラスの絞り込みは無視
      //2.クラスが指定された場合は、クラスで絞り込み
      //3.サブクラスが指定された場合は、クラスで絞り込み
      //4.クラスとサブクラスが指定された場合は、OR条件で絞り込み

      let skipFilterClasses =
        this.conditon.classes.length === 0 &&
        this.conditon.exlcudedClasses.length === 0;
      let skipFilterSubclasses = this.conditon.subclassses.length === 0;
      if (skipFilterClasses && skipFilterSubclasses) {
        return true;
      } else {
        if (skipFilterSubclasses) {
          return this.containsClasses(element);
        } else if (skipFilterClasses) {
          return this.containsSubclasses(element);
        } else {
          return (
            this.containsClasses(element) || this.containsSubclasses(element)
          );
        }
      }
    },

    contains(collection, item) {
      if (collection.length === 0) {
        return true;
      } else {
        return collection.some(e => {
          return item.includes(e);
        });
      }
    },

    doesNotContains(collection, item) {
      if (collection.length === 0) {
        return true;
      } else {
        return !collection.some(e => {
          return item.includes(e);
        });
      }
    },

    containsSubclasses(element) {
      return this.conditon.subclassses.some(e => {
        return element.subclass.includes(e);
      });
    },

    containsClasses(element) {
      // //1.クラスで絞り込む
      if (!this.contains(this.conditon.classes, element.class)) {
        return false;
      }
      // // //2.対象外クラスで絞り込む
      if (!this.doesNotContains(this.conditon.exlcudedClasses, element.class)) {
        return false;
      }
      return true;
    },

    fliterComopnents(element) {
      if (this.conditon.components.length === 0) {
        return true;
      } else {
        if (element.components.length !== this.conditon.components.length) {
          return false;
        }
        for (const s of element.components) {
          if (this.conditon.components.indexOf(s) === -1) {
            return false;
          }
        }
        return true;
      }
    },
    filterSpellname(element) {
      var trimed = this.conditon.spellname.trim();
      if (trimed === "") {
        return true;
      } else {
        //FIXME 前方一致か、includesかを指定できるようにしたい
        return element.name.includes(trimed);
        //return element.name.startsWith(trimed);
      }
    },
    //FIXME 本当はconstantsというかモデル側に移動したい。
    filterCastingTime(element) {
      if (this.conditon.casting_time === null) {
        return true;
      } else {
        if (this.conditon.casting_time !== "その他") {
          return element.casting_time.startsWith(this.conditon.casting_time);
        } else {
          let result = !(
            element.casting_time.startsWith("1ボーナスアクション") ||
            element.casting_time.startsWith("1アクション") ||
            element.casting_time.startsWith("1リアクション")
          );
          return result;
        }
      }
    },

    filterConcentration(element) {
      if (this.conditon.concentration === null) {
        return true;
      } else {
        return this.conditon.concentration === element.concentration;
      }
    },
    filterSchool(element) {
      if (this.conditon.school === null) {
        return true;
      } else {
        return element.school === this.conditon.school;
      }
    },
    //呪文本文で絞り込み
    filterDesc(element) {
      if (this.conditon.desc === null || this.conditon.desc.trim() === "") {
        return true;
      } else {
        let niniJouken = [];
        let hissuJoken = [];
        var regex = new RegExp("[ |　]?[^ ]+", "g");
        var word;
        let desc = this.conditon.desc.replace("　", " ").trim();
        // alert(desc);

        while ((word = regex.exec(desc))) {
          let str = word[0].trim();
          if (str.startsWith("|")) {
            Array.push(niniJouken, str.substring(1));
          } else {
            Array.push(hissuJoken, str);
          }
        }

        // alert(JSON.stringify(niniJouken));
        // alert(JSON.stringify(hissuJoken));
        //1.必須条件チェック
        let retVal;
        if (hissuJoken.length === 0) {
          retVal = false;
        } else {
          retVal = hissuJoken.every(desc => {
            return element.desc.includes(desc);
          });
        }

        if (retVal) {
          return true;
          //2.任意条件チェック
        } else {
          return niniJouken.some(desc => {
            return element.desc.includes(desc);
          });
        }
      }
    },
    filterTags(element) {
      if (this.conditon.tags.length === 0) {
        return true;
      } else {
        return this.conditon.tags.some(tag => {
          return tag.contains(element.name);
        });
      }
    },

    //ファイル読み込み時の処理。ファイルを読み込み、JSONデータを解析の上、呪文データを置き換える。
    onFileRead(files) {
      for (const file of files) {
        let fileReader = new FileReader();
        fileReader.onload = data => {
          let json = data.target.result;
          try {
            let spells = JSON.parse(json);
            this.spelldata = Spell.assigns(spells);
            this.showSnackbar(
              "success",
              "呪文データをファイルから読み込みました。"
            );
          } catch (e) {
            //FIXME
            this.showSnackbar("error", "呪文データの読み込みに失敗しました。");
            this.console.log(e);
          }
        };
        fileReader.readAsText(file);
      }
    },
    //テーブルセルをクリック時、expandを表示する。
    clickCell(item) {
      // props.expanded = !props.expanded;
      this.targetSpell = item;
      this.createSpell = false;
      this.showEditDialog = true;
    },
    rightClickCell(item) {
      this.addTagSpellName = item.name;
      this.showAddTagDialog = true;
    },
    //呪文詳細ダイアログから
    editTagFromDetailDalog(item) {
      this.addTagSpellName = item.name;
      this.showAddTagDialog = true;
    },
    //呪文追加ボタンを押した際、追加用のダイアログを上げる。
    addSpell() {
      this.targetSpell = null;
      this.createSpell = true;
      this.showEditDialog = true;
    },
    // //呪文編集・追加ダイアログ上で呪文を保存する。編集時は呪文データを上書きする。新規追加の場合は追加する。
    save(result) {
      if (this.targetSpell == null) {
        this.spelldata.push(result);
        this.showSnackbar("success", "呪文" + result.name + "を追加しました。");
      } else {
        Object.assign(this.targetSpell, result);
        this.targetSpell = null;
        this.showSnackbar("success", "呪文" + result.name + "を更新しました。");
      }
      this.createSpell = null;
    },
    // //呪文編集・追加ダイアログ上で呪文を削除する。編集中の呪文データを消去する。
    remove() {
      let index = this.spelldata.indexOf(this.targetSpell);
      this.spelldata.splice(index, 1);
      this.targetSpell = null;
      this.showSnackbar(
        "success",
        "呪文データをブラウザのローカルストレージから削除しました。"
      );
    },

    saveTags(updatedAllTags) {
      this.allTags = updatedAllTags;
      Tag.save(this.allTags);
      //呪文詳細ダイアログから更新された場合、以下のフラグ更新処理は不要だが、害はないので同じ処理に流し込んでいる。
      this.showAddTagDialog = false;
    },
    cancelAddTags() {
      this.addTagSpellName = null;
      this.showAddTagDialog = false;
    },
    // キャンセルもしくはクローズを押したときの挙動。
    cancelOrClose() {
      this.targetSpell = null;
      this.createSpell = null;
    },

    showHelp() {
      this.showHelpDialog = true;
    },

    //呪文データをローカルストレージに保存する
    saveSpellData() {
      localStorage.setItem("dndapp.spells", this.jesonspelldata);
      this.showSnackbar(
        "success",
        "呪文データをブラウザのローカルストレージに登録しました。"
      );
    },

    //呪文データをローカルストレージから再ロードする
    loadSpellData() {
      var spellsFromStorage = localStorage.getItem("dndapp.spells");
      //ストレージに登録されていればそのデータを復元。
      if (spellsFromStorage != undefined && spellsFromStorage != null) {
        this.spelldata = Spell.assigns(JSON.parse(spellsFromStorage));
        this.showSnackbar(
          "success",
          "ブラウザのローカルストレージに保存された呪文データを再ロードしました。"
        );
      } else {
        this.showSnackbar(
          "error",
          "ブラウザのローカルストレージに呪文データが保存されていないため、再ロードに失敗しました。"
        );
      }
    },

    //呪文データをローカルストレージから削除する。
    deleteSpellData() {
      localStorage.removeItem("dndapp.spells");
      this.spelldata = [];
      //this.spelldata = spells(); //うまく戻らない。
      this.showSnackbar(
        "success",
        "呪文データをブラウザのローカルストレージから削除しました。"
      );
    },

    formatSpellName(item) {
      var retVal = item.name + " （" + item.format(item.school, this.schools);
      if (item.ritual === "yes") {
        retVal += "、儀式";
      }
      retVal += ")";
      return retVal;
    },

    showSnackbar(level, message) {
      this.snackbar.level = level;
      this.snackbar.message = message;
      this.snackbar.show = true;
    },

    closeMobileSearchDetailDialog() {
      this.showMobileSearchDetailDialog = false;
    },

    //音声認識
    //@see https://qiita.com/Sa2Knight/items/a7deb5b5d07820f6f19e
    //@see https://developer.mozilla.org/ja/docs/Web/API/Web_Speech_API
    //@see https://www.cresco.co.jp/blog/entry/9035/
    getSpeechRecognition() {
      const SpeechRecogintion = window.webkitSpeechRecognition;
      if (SpeechRecogintion) {
        let retVal = new SpeechRecogintion();
        // retVal.lang = "en-US";
        retVal.lang = "ja-JP";
        //grammerは利用できない模様。削除を検討中の模様。
        //https://github.com/WICG/speech-api/issues/57
        //https://github.com/WICG/speech-api/pull/58
        // retVal.continuous = true;
        // var grammar =
        //   "#JSGF V1.0 JIS ja; grammar numbers; public <number> = 10 | 20 | 30 | 40 | 50 | 60 | 70 | 80 | 90 | 100 ;";
        // var grammar2 =
        //   "#JSGF V1.0 JIS ja; grammar words; public <word> = きろ | もうろう;";
        // const speechRecognitionList = new window.webkitSpeechGrammarList();
        // speechRecognitionList.addFromString(grammar, 0.1);
        // speechRecognitionList.addFromString(grammar2, 1.0);
        // retVal.grammars = speechRecognitionList;
        // retVal.maxAlternatives = 5;
        return retVal;
      } else {
        return null;
      }
    }
  }
};
</script>
