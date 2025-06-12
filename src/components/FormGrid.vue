<template>
    <div class="form-group">
      <div :class="classList">
        <div :id="uniqIdsMixin" ></div>
      </div>
      
      <div class="grid-cnt">
        <table :id="'grid-element_'+ name" class="display"></table>
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
import { createUniqIdsMixin } from 'vue-uniq-ids'
import DataFormatMixin from './mixins/DataFormat';
import OptionboxView from "./FormSelectList/OptionboxView";
import Mustache from "mustache";
import ValidationMixin from "./mixins/validation";
import DataTable from 'datatables.net-dt';
import 'datatables.net-select';
import { name } from "mustache";

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
      "isSearchEnable",
      "isPaginationEnable",
      "pageSize",
      "validationData",
      "label",
      "renderVarHtml",
      "footerCallback"
    ],
    data() {
      return {
        counter:255555,
        originalEscapeFn: null,
        customFunctions: {}
      };
    },
    mounted(){
        var data;
        var columns;

        if(this.value !== undefined && this.value !== null){
          if(this.column !== undefined && this.column.length > 0){
            data=eval(this.value);
            columns=eval(this.column);
            
          }else{
            data=eval(this.value).data;
            columns=eval(this.value).columns;
          }
        }

        this.generateGrid(data,columns);              
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
        // this.overwriteMustacheEscape();
        // try {
         
        //   if (this.renderVarHtml) {
        //     return Mustache.render(this.content, data);
        //   }
        //   return Mustache.render(this.content, data);
        // } catch (error) {
        //   if (this.renderVarHtml) {
        //     return this.renderVarName;
        //   }
        //   return this.content;
        // } finally {
        //   Mustache.escape = this.originalEscapeFn;
        // }
      }
    },
    methods: {
      generateGrid(inputData,inputColumns){
        // console.log("this.name:"+this.name +"-------------this.isSearchEnable:"+this.isSearchEnable+"-----------------------this.isPaginationEnable:"+this.isPaginationEnable+"--------------------this.pageSize:"+this.pageSize+"-----------------------------this.footerCallback:"+this.footerCallback+"-----------------data:"+inputData+"---------------------columns:"+inputColumns)
        
        var table = new DataTable('#grid-element_' + this.name);
        table.destroy();
        $('#grid-element_' + this.name).empty();


        if(this.footerCallback !== undefined && this.footerCallback.length > 0){
            var table = document.querySelector('#grid-element_' + this.name);
            if (!table.querySelector('tfoot')) {
              var tfoot = document.createElement('tfoot');
              var tr = document.createElement('tr');

              // Number of columns = length of your dynamic columns array
              var colCount = inputColumns.length;

              // Create empty footer cells matching columns
              for (var i = 0; i < colCount; i++) {
                  var th = document.createElement('th');
                  tr.appendChild(th);
              }

              tfoot.appendChild(tr);
              table.appendChild(tfoot);
            }
        }


        var ddd=new DataTable('#grid-element_' + this.name, {
                      data: inputData,//eval(newVal), //JSON.parse(newVal),
                      columns: inputColumns,
                      responsive: true,
                      destroy:true,
                      orderMulti:true,
                      scrollX: true,
                      select :"multi",
                      searching: this.isSearchEnable ?? false,
                      paging: this.isPaginationEnable ?? false,
                      pageLength:this.pageSize ?? 20,
                      footerCallback:eval('(' + this.footerCallback + ')')
                  });        
        
      },
      registerCustomFunction(name, implementation) {
        this.customFunctions[name] = implementation;
      },
    },
    watch: { 
        "value": function(newVal, oldVal) { // watch it

            var data;
            var columns;

            
            if(this.column !== undefined && this.column.length > 0){
              data=eval(newVal);
              columns=eval(this.column);
              
            }else{
              data=eval(newVal).data;
              columns=eval(newVal).columns;
            }
          

            this.generateGrid(data,columns);
        },
        "column": function(newVal, oldVal) { // watch it
          
          var data;
            var columns;

            
            if(newVal !== undefined && newVal.length > 0){
              data=eval(this.value);
              columns=eval(newVal);
              
            }else{
              data=eval(newVal).data;
              columns=eval(newVal).columns;
            }
          

            this.generateGrid(data,columns);
        }
    }
  };
  </script>

  

<style lang="scss">
@import 'datatables.net-dt/css/dataTables.dataTables.css';
</style>