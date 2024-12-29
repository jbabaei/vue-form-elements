<template>
    <div class="form-group">
      <!-- <label :class="labelClass" v-uni-for="name">{{ $t(label) }}</label> -->

      <div :class="classList">
        <div :id="uniqIdsMixin" ></div>
      </div>
      
      <!-- Data
      <div v-html="content"></div>

      
      <input type="hidden" :value="JSON.stringify(value)"> -->
      
      <div class="grid-cnt">
        <table id="grid-element" class="display"></table>
      </div>
      <div
        v-if="(validator && validator.errorCount) || error"
        class="invalid-feedback"
      >
        <div
          v-for="(error, index) in validator.errors.get(this.name)"
          :key="index"
        >
          {{ error }}
        </div>
        <div v-if="error">{{ error }}</div>
      </div>
      <small v-if="helper" class="form-text text-muted">{{ helper }}</small>
    </div>
  </template>
  
  <script>
  import { createUniqIdsMixin } from "vue-uniq-ids";
  import DataFormatMixin from './mixins/DataFormat';
  import OptionboxView from "./FormSelectList/OptionboxView";
  import Mustache from "mustache";
  import ValidationMixin from "./mixins/validation";
//   import Editor from "./Editor";
  import { formatIfDate } from "../dateUtils";
  import DataTable from 'datatables.net-dt';

  // Create the mixin
  const uniqIdsMixin = createUniqIdsMixin();
  
  export default {
    components: {
      OptionboxView
    },
    mixins: [uniqIdsMixin, ValidationMixin,DataFormatMixin],
    inheritAttrs: false,
    props: [
      "error",
      "name",
      "helper",
      "value",
      "controlClass",
      //"content",
      "column",
      "validationData",
      "label",
      "renderVarHtml"
    ],
    data() {
      return {
        counter:255555,
        originalEscapeFn: null,
        customFunctions: {},
        //editorSettings: {
        //  inline: true,
        //  menubar: false,
        //  plugins: ["link", "lists"],
        //  toolbar: `undo redo | link | styleselect | bold italic forecolor |
        //      alignleft aligncenter alignright alignjustify | bullist numlist outdent indent`,
        //   skin: false,
        //   relative_urls: false,
        //   convert_urls: false,
        //   remove_script_host: false
        // }
      };
    },
    mounted(){
        //console.log("formGrid rendered--------------",this.value);
          
          if(this.value !== undefined && this.value !== null){
            if(this.column !== undefined && this.column.length > 0){
              var ddd=new DataTable('#grid-element', {
                        data: eval(this.value),//eval(newVal), //JSON.parse(newVal),
                        columns: eval(this.column),
                        responsive: true,
                        destroy:true,
                        orderMulti:true,
                        scrollX: true
                    });
            }else{
              //var ddd=new DataTable('#grid-element', this.value);//eval(newVal));
              
              var ddd=new DataTable('#grid-element', {
                        data: eval(this.value).data,//eval(newVal), //JSON.parse(newVal),
                        columns: eval(this.value).columns,
                        responsive: true,
                        destroy:true,
                        orderMulti:true,
                        scrollX: true
                    });//eval(newVal));
            }
          }
      },
    computed: {
      classList() {
        const classList = {
          "is-invalid":
            (this.validator && this.validator.errorCount) || this.error
        };
        if (this.controlClass) {
          classList[this.controlClass] = true;
        }
        return classList;
      },
      sourceConfig() {
        return {
            dataSource: this.options.dataSource,
            collectionOptions: this.options.collectionOptions,
            selectedEndPoint: this.options.selectedEndPoint,
            selectedDataSource: this.options.selectedDataSource,
            valueTypeReturned: this.options.valueTypeReturned,
            dataName: this.options.dataName,
            value: this.options.value,
            key: this.options.key
          };
      },
      rendered() {
        // If we have't validationData, we can't evaluate the mustache variables
        // Used by ScreenBuilder in Design Mode
        if (!this.validationData) {
          return this.content;
        }
        const data = this.makeProxyData(); // Gets the data
        this.overwriteMustacheEscape();
        try {
         
          if (this.renderVarHtml) {
            return Mustache.render(this.content, data);
          }
          return Mustache.render(this.content, data);
        } catch (error) {
          if (this.renderVarHtml) {
            return this.renderVarName;
          }
          return this.content;
        } finally {
          Mustache.escape = this.originalEscapeFn;
        }
      }
    },
    methods: {
      // generateGrid(){
      //   alert("generate...")

      //   var ddd=new DataTable('.grid-element', JSON.parse(this.content));
      // },
      /**
       * Backup and overwrite the original mustache escaped property
       */
      overwriteMustacheEscape() {
        this.originalEscapeFn = Mustache.escape;
        Mustache.escape = this.mustacheEscapeFn;
      },
      /**
       * Register custom functions to be included
       * @param {string} name
       * @param {object} implementation
       */
      registerCustomFunction(name, implementation) {
        this.customFunctions[name] = implementation;
      },
      /**
       * Escape the mustache code, added in the tinyMCE editor
       * @param {string} text
       * @return {object}
       */
      mustacheEscapeFn(text) {
        const formatedText = formatIfDate(text);
        if (this.renderVarHtml) {
          return formatedText;
        }
        return this.originalEscapeFn(formatedText);
      }
    },
    watch: { 
        "value": function(newVal, oldVal) { // watch it
          // console.log('Prop changed: ', newVal, ' | was: ', oldVal);

          // console.log('column value---',this.column);

          if(this.column !== undefined && this.column.length > 0){
            var ddd=new DataTable('#grid-element', {
                      data: eval(newVal),//eval(newVal), //JSON.parse(newVal),
                      columns: eval(this.column),
                      responsive: true,
                      destroy:true,
                      orderMulti:true,
                      scrollX: true
                  });
          }else{
            //var ddd=new DataTable('#grid-element', newVal);//eval(newVal));
            var ddd=new DataTable('#grid-element', {
                      data: eval(newVal).data,//eval(newVal), //JSON.parse(newVal),
                      columns: eval(newVal).columns,
                      responsive: true,
                      destroy:true,
                      orderMulti:true,
                      scrollX: true
                  });
          }

          // console.log("dataTable -----",ddd);
        },
        "column": function(newVal, oldVal) { // watch it
          // console.log('Column Prop changed: ', newVal, ' | was: ', oldVal);

          // console.log("+++ Value:",this.value);

          if(newVal !== undefined && newVal.length > 0){
            var ddd=new DataTable('#grid-element', {
                      data: eval(this.value),//eval(newVal), //JSON.parse(newVal),
                      columns: eval(newVal),
                      responsive: true,
                      destroy:true,
                      orderMulti:true,
                      scrollX: true
                  });
          }else{
            //var ddd=new DataTable('#grid-element', newVal);//eval(newVal));

            var ddd=new DataTable('#grid-element', {
                      data: eval(newVal).data,//eval(newVal), //JSON.parse(newVal),
                      columns: eval(newVal).columns,
                      responsive: true,
                      destroy:true,
                      orderMulti:true,
                      scrollX: true
                  });//eval(newVal));
          }


          // var ddd=new DataTable('#grid-element', {
          //           data: eval(this.value), //this.value,//eval(this.content), //JSON.parse(this.value),
          //           columns: eval(newVal),
          //           responsive: true,
          //           destroy:true,
          //           orderMulti:true
          //       });

                // console.log("dataTable -----",ddd);
        }
    }
  };
  </script>
  
  <style scoped>
  .invalid-feedback {
    display: block;
  }
  
  .is-invalid {
    border: 1px solid #dc3545;
    border-radius: 0.25rem;
  }

  @media only screen and (max-width: 990px) {
    table.dataTable thead>tr>th,table.dataTable tbody>tr>td{
      min-width: 140px;
    }
  }
  </style>
  

<style lang="scss">
@import 'datatables.net-dt/css/dataTables.dataTables.css';
</style>