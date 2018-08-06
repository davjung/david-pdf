<template>
  <div class="pdf-viewer">
    <header class="pdf-viewer__header box-shadow">
      <!-- <div class="pdf-preview-toggle"><a @click.prevent.stop="togglePreview" class="icon"><PreviewIcon /></a></div> -->
      <span>
        <PDFZoom :scale="scale" @change="updateScale" @fit="updateFit" class="header-item"/> 
        <PDFPaginator v-model="currentPage" :pageCount="pageCount" class="header-item"/>

        <slot name="header"></slot>
        
        <button v-if="this.currentPage>1" class="buttontest" @click="prev()">prev</button>
        <button v-if="this.currentPage<this.pageCount" class="buttontest" @click="next()">next</button>
      </span>

      <span class="buttons">
        <button v-if="this.buttons[0] != 1" @click="prevPages()">left</button>
        <button v-for="page in this.buttons" :key="page.id" @click="gotoPage(page)">{{page}}</button>
        <button v-if="this.pageCount != this.buttons.slice(-1)[0]" @click="nextPages()">right</button>
      </span>
      

    </header>

    <PDFData 
      class="pdf-viewer__main" 
      :url="url"
      :currentPage="currentPage" 
      @page-count="updatePageCount" 
      @page-focus="updateCurrentPage" 
      @document-rendered="onDocumentRendered" 
      @document-errored="onDocumentErrored"
    >
      <PDFPreview
        slot="preview"
        slot-scope="{pages}"
        v-show="isPreviewEnabled"
        class="pdf-viewer__preview"
        v-bind="{pages, scale, currentPage, pageCount, isPreviewEnabled}"
        />
      <PDFDocument
        class="pdf-viewer__document"
        :class="{ 'preview-enabled': isPreviewEnabled }"
        slot="document"
        slot-scope="{pages}"
        v-bind="{pages, scale, optimalScale, fit, currentPage, pageCount, isPreviewEnabled}"
        @scale-change="updateScale"
        />


    </PDFData>

    <!-- <canvas id="the-canvas" style="border:1px solid black"></canvas> -->
    
  </div>
</template>

<script>
import PreviewIcon from '../assets/icon-preview.svg';

import PDFDocument from './PDFDocument';
import PDFData from './PDFData';
import PDFPaginator from './PDFPaginator';
import PDFPreview from './PDFPreview';
import PDFZoom from './PDFZoom';

function floor(value, precision) {
  const multiplier = Math.pow(10, precision || 0);
  return Math.floor(value * multiplier) / multiplier;
}


export default {
  name: 'PDFViewer',

  components: {
    PDFDocument,
    PDFData,
    PDFPaginator,
    PDFPreview,
    PDFZoom,
    PreviewIcon,
  },

  props: {
    url: String,
  },

  data() {
    return {
      scale: undefined,
      optimalScale: undefined,
      fit: undefined,
      currentPage: 1,
      pageCount: undefined,
      isPreviewEnabled: false,
      buttons: [],
    };
  },


  methods: {
    initButtons() {
      let j;
      if (this.pageCount <= 10) j = this.pageCount;
      else j = 10;
        
      for(var i = 1; i<=j; i++) {
        this.buttons.push(i)
      }
    },

    adjustButtons() {
      let x = Math.floor(this.currentPage / 10) * 10 + 1;
      let y = x + 9;
      let arr = [];
      for (x; x<y; x++) arr.push(x); 
      this.buttons = arr;
    },


    gotoPage(page) {
      if (page > 0 && page < this.pageCount) {
        this.currentPage = page;
        this.adjustButtons();
      }
    },

    prevPages() {
      let i = this.buttons[0] - 10;
      let arr = [];
      for (i; i<this.buttons[0]; i++) {
        arr.push(i);
      }
      this.buttons = arr;
    },

    nextPages() {
      let i = this.buttons.slice(-1)[0] + 1;
      let j = Math.min(this.pageCount, i+9);

      let arr = [];
      for (i; i<=j; i++){
        arr.push(i);
      }
      this.buttons = arr;
    },

    prev() {
      if (this.currentPage > 1) {
        this.currentPage--;
        if (this.currentPage < this.buttons[0]) this.prevPages()
      }
    }, 
    next() {
      if(this.currentPage < this.pageCount) {
        this.currentPage++;
      }
    },
    onDocumentRendered() {
      this.$emit('document-errored', this.url);
    },

    onDocumentErrored(e) {
      this.$emit('document-errored', e);
    },

    updateScale({scale, isOptimal = false}) {
      const roundedScale = floor(scale, 2);
      if (isOptimal) this.optimalScale = roundedScale;
      this.scale = roundedScale;
    },

    updateFit(fit) {
      this.fit = fit;
    },

    updatePageCount(pageCount) {
      this.pageCount = pageCount;
    },

    updateCurrentPage(pageNumber) {
      this.currentPage = pageNumber;
    },

    togglePreview() {
      this.isPreviewEnabled = !this.isPreviewEnabled;
    },
  },

  watch: {
    url() {
      this.currentPage = undefined;
    },
  },

  mounted() {
    document.body.classList.add('overflow-hidden');
    this.initButtons();
  },
};
</script>

<style scoped>
header span{
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  padding: 1em;
  position: relative;
  z-index: 99;
}



.header-item {
  margin: 0 2.5em;
}

.pdf-viewer .pdf-viewer__document,
.pdf-viewer .pdf-viewer__preview {
  top: 150px;
}

.pdf-viewer__preview {
  display: block;
  width: 15%;
  right: 85%;
}

.pdf-viewer__document {
  top: 70px;
  width: 100%;
  left: 0;
}

.pdf-viewer__document.preview-enabled {
  width: 85%;
  left: 15%;

}

@media print {
  header {
    display: none;
  }
}

.buttontest {
  z-index: 1000;
}

.buttons {
  position: relative;
  bottom: 15px;
}
</style>
