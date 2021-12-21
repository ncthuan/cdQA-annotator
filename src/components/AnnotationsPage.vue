<template>
  <div class="AnnotationsPage container mt-3">
    <div>
      <b-navbar sticky toggleable="lg" class="pl-0">
        <b-navbar-brand>cdQA-annotator</b-navbar-brand>

        <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

        <b-collapse id="nav-collapse" is-nav>
          <!-- Right aligned nav items -->
          <b-navbar-nav class="ml-auto">
            <!--
            <b-nav-item>
              <svg-progress-bar :value="data_number / json.data.length * 100" :options="options"></svg-progress-bar>
            </b-nav-item>
            -->
            <b-nav-form>
              <div style="margin-right: 8px">
                <b-button :variant="'outline-secondary'" v-on:click="prev">Previous</b-button>
                <b-button :variant="'outline-secondary'" v-on:click="next">Next</b-button>
              </div>
              <vue-bootstrap-typeahead
                v-model="query"
                :data="autocomplete"
                placeholder="Search a document..."
                @hit="data_number = autocomplete.indexOf($event); context_number = 0"
              />
            </b-nav-form>
            <b-nav-item right>
              <!-- v-on:click="delete_empty_document()" v-download-data:type="'text/json'" -->
              <b-button
                :variant="'primary'"
                v-download-data="valid_json"
                v-download-data:filename="'export.json'"
              >Download</b-button>
            </b-nav-item>
          </b-navbar-nav>
        </b-collapse>
      </b-navbar>
    </div>
    <br>

    <div>
      <h2>{{ json.data[data_number].title }}</h2>
      <span
        class="text-muted"
      >Paragraph {{ context_number+1 }} of {{ json.data[data_number].paragraphs.length }} | Document {{ data_number+1 }} of {{ json.data.length }}</span>
      <br>
      <br>

      <p style="white-space: pre-line;" ref="paragraph" v-selection.fix="{getSelection:getSelection}">{{ paragraph_context }}</p>
      <br>

      <b-form-input v-model="question" type="text" placeholder="Type question here..."></b-form-input>
      <br>

      <b-form-input v-model="answer" type="text" disabled placeholder="answer..."></b-form-input>
      <br>

      <b-form-input v-model="answer_start" type="text" disabled placeholder="answer start position..."></b-form-input>
      <br>

      <b-button :size="''" :variant="'secondary'" v-on:click="addAnnotation()">Add annotation</b-button>
      <!--
      <b-button
        v-if="context_number < json.data[data_number].paragraphs.length"
        :size="''"
        :variant="'danger'"
        v-on:click="delete_paragraph()"
      >Delete paragraph</b-button>
      <b-button
        v-else
        :size="''"
        :variant="'danger'"
        v-on:click="delete_paragraph(), data_number += 1, context_number = 0"
      >Delete paragraph</b-button>
      -->
      <br>
      <br>

      <b-table striped hover :items="items" :fields="fields">
        <template slot="Edit" slot-scope="row">
          <b-button :size="''" :variant="'danger'" @click.stop="deleteAnnotation(row.index)">Delete</b-button>
        </template>
      </b-table>
      <br>

    </div>
  </div>
</template>

<script>

const uuidv4 = require('uuid/v4');

export default {
  name: "AnnotationsPage",
  props: ["json"],
  data: function() {
    return {
      data_number: 0,
      context_number: 0,
      question: "",
      answer: "",
      answer_start: "",
      fields: ["Questions", "Answers", "Edit"],
      query: ""
    };
  },
  methods: {
    addAnnotation: function() {
      var paragraph_container = this.json.data[this.data_number].paragraphs[
        this.context_number - 1
      ];
      var qa = {
        question: this.question,
        id: uuidv4(),
        answers: [{ answer_start: this.answer_start, text: this.answer }]
      };
      paragraph_container.qas.push(qa);
      this.question = "";
      this.answer = "";
    },
    deleteAnnotation: function(row_index) {
      var paragraph_container = this.json.data[this.data_number].paragraphs[this.context_number];
      paragraph_container.qas.splice(row_index, 1);
    },
    getSelection: function() {
      const selection = window.getSelection();
      this.answer_start = selection.baseOffset;
      this.answer = selection.toString();
    },
    delete_paragraph: function() {
      var paragraph_container = this.json.data[this.data_number].paragraphs;
      paragraph_container.splice([this.context_number], 1);
    },
    delete_empty_document: function() {
      for (var i in this.json.data) {
        if (this.json.data[i].paragraphs.length == 0) {
          this.json.data.splice(i, 1);
        }
      }
    },
    next: function() {
      this.context_number += 1;
      if (this.context_number >= this.json.data[this.data_number].paragraphs.length) {
        this.context_number = 0;
        if (this.data_number < this.json.data.length-1) this.data_number += 1;
      }
    },
    prev: function() {
      this.context_number -= 1;
      if (this.context_number <= 0) {
        this.context_number = 0;
        if (this.data_number > 0) this.data_number -= 1;
      }
    }
  },
  computed: {
    valid_json: function() {
      var json = JSON.stringify(this.json, undefined, 2)
        .replace(/[\u007F-\uFFFF]/g, function(chr) {
          return "\\u" + ("0000" + chr.charCodeAt(0).toString(16)).substr(-4);
        });
      return json;
    },
    autocomplete: function() {
      var idx = [];
      for (var i = 0; i < this.json.data.length; i++) {
        idx.push(this.json.data[i].title);
      }
      return idx;
    },
    items: function() {
      var paragraph_container = this.json.data[this.data_number].paragraphs[this.context_number];
      var items = [];
      for (var i = 0; i < paragraph_container.qas.length; i++) {
        var item = {
          Questions: paragraph_container.qas[i].question,
          Answers: paragraph_container.qas[i].answers[0].text
        };
        items.push(item);
      }
      return items;
    },
    paragraph_context: function() {
      return this.json.data[this.data_number].paragraphs[this.context_number].context;
    },
    options() {
      return {
        radius: 15,
        circleLineCap: "round",
        varyStrokeArray: [1, 2],
        pathColors: ["lightgrey", "rgb(40, 167, 69)"],
        text: function(value) {
          return (
            this.htmlifyNumber(value) + '<span style="font-size: 1em;">%</span>'
          );
        },
        textColor: "black"
      };
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
