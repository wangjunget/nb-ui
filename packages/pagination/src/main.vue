<template>
  <div
    class="nb-pagination"
    onselectstart="return false"
  >
    <span
      v-if="showTotal"
      class="nb-pagination__total"
    >共 {{ total }} 条</span>

    <nb-button
      type="text"
      class="nb-pagination__btn nb-pagination__prevbtn"
      :text="prevText"
      :disabled="innerCurrentPage<=1"
      @click="handlePageChange('left')"
    />

    <nb-pages
      :current-page.sync="innerCurrentPage"
      :page-count="pageCount"
      :page-size="pageSize"
      :pages="pages"
      :total="total"
      v-bind="$attrs"
    />

    <nb-button
      type="text"
      class="nb-pagination__btn nb-pagination__nextbtn"
      :text="nextText"
      :disabled="innerCurrentPage>=Math.ceil(total/pageSize)"
      @click="handlePageChange('right')"
    />

    <div
      v-if="showJump"
      class="nb-pagination-jump"
    >
      <input
        v-model="inputPageNum"
        min="1"
        :max="pageTotal"
        type="number"
        class="nb-pagination__jumpipt"
        @keyup="handleKeyup"
      >
      <nb-button
        type="text"
        class="nb-pagination__jumpbtn"
        text="跳转"
        @click="handlePageJump"
      />
    </div>
  </div>
</template>

<script type="text/babel">
  import nbPages from './pages'
  import nbButton from '../../button/src/main'
  export default {
    name: 'NbPagination',
    components: {
      nbPages,
      nbButton
    },
    props: {
      total: {
        type: Number,
        default: 0
      },
      currentPage: {
        type: Number,
        default: 1
      },
      pageSize: {
        type: Number,
        default: 10
      },
      pageCount: {
        type: Number,
        validator (value) {
          return (value | 0) === value && value > 6 && value < 22 && (value % 2) === 1;
        },
        default: 7
      },
      prevText: {
        type: String,
        default: '<'
      },
      nextText: {
        type: String,
        default: '>'
      },
      showJump: {
        type: Boolean,
        default: false,
      },
      showTotal: {
        type: Boolean,
        default: false
      } ,
    },

    data() {
      return {
        pages: [],
        pageTotal: 1,
        innerCurrentPage: this.currentPage,

        inputPageNum: this.currentPage,
      };
    },

    computed: {
      
    },

    watch:{
      innerCurrentPage(val) {
        this.$parent.currentPage = val;
        this.inputPageNum = val;
        this.getPages(val);
        this.$emit('currentChange', val);
        this.$emit('prevClick', val);
        this.$emit('nextClick', val);
      }
    },

    created () {
      this.getPages(this.currentPage)
    },

    methods: {
      handlePageChange (type) {
        if (type==='left') {
          this.innerCurrentPage--;
        }else if (type==='right') {
          this.innerCurrentPage++;
        }
      },

      getPages (val) {
        const pageTotal = Math.ceil(this.total/this.pageSize);
        this.pageTotal = pageTotal;
        // 总页数小于设置的
        let pageArr = [];
        if(pageTotal <= this.pageCount){
          for (let i=0; i<pageTotal; i++ ) {
            pageArr.push(i+1)
          }
        }else{
          let startPage = val - parseInt(this.pageCount/2);
          let endPage = val + parseInt(this.pageCount/2);
          startPage = startPage<1 ? 1 : startPage;
          endPage = endPage>pageTotal ? pageTotal : endPage;
          if (startPage === 1) {
            for (let i=0; i<this.pageCount; i++) {
              pageArr.push(i+1);
            }
            pageArr[this.pageCount-2] = '···';
            pageArr[this.pageCount-1] = pageTotal;
          }else if (endPage === pageTotal) {
            for (let i=0; i<this.pageCount; i++) {
              pageArr.push(pageTotal-this.pageCount+i+1);
            }
            pageArr[0] = 1;
            pageArr[1] = '···';
          }else{
            for (let i=0; i<this.pageCount; i++) {
              pageArr.push(startPage+i);
            }
            pageArr[0] = 1;
            pageArr[1] = '···';
            pageArr[this.pageCount-2] = '···';
            pageArr[this.pageCount-1] = pageTotal;
          }
        }
        this.pages = pageArr;
      },

      handleKeyup({ keyCode }) {
        if (keyCode === 13) {
          this.handlePageJump();
        }
      },

      handlePageJump () {
        this.innerCurrentPage = this.validCurrentPage(this.inputPageNum);
      },



      validCurrentPage (val) {
        val = parseInt(val, 10);
        const havePageCount = typeof this.pageTotal === 'number';
        let resetValue;
        if (!havePageCount) {
          if (isNaN(val) || val < 1) resetValue = 1;
        } else {
          if (val < 1) {
            resetValue = 1;
          }else if (val > this.pageTotal) {
            resetValue = this.pageTotal;
            this.inputPageNum = this.pageTotal;
          }
        }
        if (resetValue === undefined && isNaN(val)) {
          resetValue = 1;
        }else if (resetValue === 0) {
          resetValue = 1;
        }
        return resetValue === undefined ? val : resetValue;
      }
    }
  };
</script>

<style lang="scss" src="../../../theme/pagination.scss"></style>
