<template>
  <ul class="c-pagination">
    <li v-if="hasPreviousPages" class="c-pagination__item">
      <button type="button" :data-page-number="0 + 1" class="c-pagination__button" @click="handlePageChange({ action: ACTIONS.FIRST })">
        &lt;&lt;
      </button>
    </li>

    <li v-if="hasPreviousPage" class="c-pagination__item">
      <button
        type="button"
        class="c-pagination__button"
        :data-page-number="currentPageNumber - 1"
        @click="handlePageChange({ action: ACTIONS.PREVIOUS })"
      >
        &lt;
      </button>
    </li>

    <li v-for="pageNumber in pageCountArray" :key="pageNumber">
      <button
        ref="numberButtons"
        type="button"
        :class="['c-pagination__button', 'js-pagination-number', value + pageNumber - 1 == value ? CLASSES.IS_ACTIVE : null]"
        :data-page-number="pageNumber"
        @click="handlePageChange({ element: $event.target })"
      >
        {{ pageNumber }}
      </button>
    </li>

    <li v-if="hasNextPage" class="c-pagination__item">
      <button
        type="button"
        :data-page-number="currentPageNumber + 1"
        class="c-pagination__button"
        @click="handlePageChange({ action: ACTIONS.NEXT })"
      >
        &gt;
      </button>
    </li>

    <li v-if="hasNextPages" class="c-pagination__item">
      <button
        type="button"
        :data-page-number="totalPageCount - 1"
        class="c-pagination__button"
        @click="handlePageChange({ action: ACTIONS.LAST })"
      >
        &gt;&gt;
      </button>
    </li>
  </ul>
</template>

<script>
export default {
  props: {
    totalPageCount: {
      type: Number,
      default: 1,
    },
    maxVisiblePageCount: {
      type: Number,
      default: 5,
    },
    value: {
      type: Number,
      default: 1,
    },
  },

  data() {
    return {
      NODE_DATASETS: {
        PAGE_NUMER: 'data-page-number',
      },

      CLASSES: {
        IS_ACTIVE: 'is-active',
      },

      ACTIONS: {
        FIRST: 'first',
        PREVIOUS: 'previous',
        NEXT: 'next',
        LAST: 'last',
      },

      hasNextPage: false,
      hasNextPages: false,

      hasPreviousPage: false,
      hasPreviousPages: false,

      pageCountArray: [],

      visiblePageCounts: 0,
      currentPageNumber: 1,
    };
  },

  watch: {
    value() {
      this.currentPageNumber = this.value;
      this.handlePageChange({ elementPageNumber: this.value });
    },
  },

  mounted() {
    this.handleVisiblePageCounts(this.value);
    this.currentPageNumber = this.value;
  },

  methods: {
    handleVisiblePageCounts(pageNumber) {
      this.pageCountArray = [];

      for (
        let i = pageNumber - Math.floor(this.maxVisiblePageCount / 2);
        i < pageNumber + Math.floor(this.maxVisiblePageCount / 2) + 1;
        i++
      ) {
        if (i > 0 && i <= this.totalPageCount) {
          this.pageCountArray.push(i);
        }
      }

      if (this.pageCountArray.includes(this.totalPageCount)) {
        this.hasNextPages = false;
      } else {
        this.hasNextPages = true;
      }

      if (this.pageCountArray.includes(1)) {
        this.hasPreviousPages = false;
      } else {
        this.hasPreviousPages = true;
      }
    },

    handleClasses(params) {
      const { element, elementPageNumber } = params;

      this.$refs.numberButtons.forEach(currentItem => {
        currentItem.classList.remove(this.CLASSES.IS_ACTIVE);
      });

      this.$nextTick(() => {
        if (element) {
          element.classList.add(this.CLASSES.IS_ACTIVE);
        } else {
          const selectedElement = this.$el.querySelector(`[${this.NODE_DATASETS.PAGE_NUMER}="${elementPageNumber}"].js-pagination-number`);
          selectedElement.classList.add(this.CLASSES.IS_ACTIVE);
        }
      });
    },

    handleMoreButtons(params) {
      const { elementPageNumber } = params;

      if (elementPageNumber == this.totalPageCount) {
        this.hasNextPages = false;
        this.hasNextPage = false;
      } else {
        this.hasNextPages = true;
        this.hasNextPage = true;
      }

      if (elementPageNumber == 1) {
        this.hasPreviousPages = false;
        this.hasPreviousPage = false;
      } else {
        this.hasPreviousPages = true;
        this.hasPreviousPage = true;
      }
    },

    handlePageChange(params) {
      let { element, action, elementPageNumber } = params;

      if (element) {
        elementPageNumber = parseInt(element.getAttribute(this.NODE_DATASETS.PAGE_NUMER), 10);
      } else {
        elementPageNumber = this.currentPageNumber;
      }

      if (element && this.currentPageNumber == elementPageNumber) {
        return;
      }

      if (action == this.ACTIONS.NEXT) {
        if (elementPageNumber == this.totalPageCount) {
          return;
        }

        elementPageNumber += 1;
      } else if (action == this.ACTIONS.PREVIOUS) {
        if (elementPageNumber == 1) {
          return;
        }

        elementPageNumber -= 1;
      } else if (action == this.ACTIONS.FIRST) {
        elementPageNumber = 1;
      } else if (action == this.ACTIONS.LAST) {
        elementPageNumber = this.totalPageCount;
      }

      this.handleMoreButtons({ elementPageNumber });
      this.handleVisiblePageCounts(elementPageNumber);
      this.handleClasses({ element, elementPageNumber });

      this.$emit('input', elementPageNumber);
      this.currentPageNumber = elementPageNumber;
    },
  },
};
</script>

<style lang="scss" scoped>
@import './Pagination';
</style>
