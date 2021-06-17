<template>
  <ul class="c-pagination">
    <li v-if="hasPreviousPages" class="c-pagination__item">
      <button
        type="button"
        :data-page-number="0 + 1"
        class="c-pagination__button"
        @click="handlePageChange({ action: ACTIONS.FIRST })"
      >
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
        :class="[
          'c-pagination__button',
          'js-pagination-number',
          currentPage + pageNumber - 1 == currentPage ? CLASSES.IS_ACTIVE : null,
        ]"
        :data-page-number="currentPage + pageNumber - 1"
        @click="handlePageChange({ element: $event.target })"
      >
        {{ currentPage + pageNumber - 1 }}
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
    currentPage: {
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
    currentPage() {
      this.handleVisiblePageCounts(this.currentPage);
    },
    totalPageCount() {
      this.handleVisiblePageCounts(this.currentPage);
    },
  },

  mounted() {
    this.handleVisiblePageCounts(this.currentPage);
    this.currentPageNumber = this.currentPage;
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

    handlePageChange(params) {
      const { element, action } = params;

      let elementPageNumber = null;

      if (element) {
        elementPageNumber = parseInt(element.getAttribute(this.NODE_DATASETS.PAGE_NUMER), 10);
      } else {
        elementPageNumber = this.currentPageNumber;
      }

      if (element && this.currentPageNumber == elementPageNumber) {
        return;
      }

      if (action === this.ACTIONS.NEXT) {
        if (elementPageNumber === this.totalPageCount) {
          return;
        }

        elementPageNumber += 1;
      } else if (action === this.ACTIONS.PREVIOUS) {
        if (elementPageNumber === 1) {
          return;
        }

        elementPageNumber -= 1;
      } else if (action === this.ACTIONS.FIRST) {
        elementPageNumber = 1;
      } else if (action === this.ACTIONS.LAST) {
        elementPageNumber = this.totalPageCount;
      }

      if (elementPageNumber === this.totalPageCount) {
        this.hasNextPages = false;
        this.hasNextPage = false;
      } else {
        this.hasNextPages = true;
        this.hasNextPage = true;
      }

      if (elementPageNumber === 1) {
        this.hasPreviousPages = false;
        this.hasPreviousPage = false;
      } else {
        this.hasPreviousPages = true;
        this.hasPreviousPage = true;
      }

      this.handleVisiblePageCounts(elementPageNumber);

      this.$refs.numberButtons.forEach(currentItem => {
        currentItem.classList.remove(this.CLASSES.IS_ACTIVE);
      });

      this.$nextTick(() => {
        if (element) {
          element.classList.add(this.CLASSES.IS_ACTIVE);
        } else {
          const element2 = this.$el.querySelector(
            `[${this.NODE_DATASETS.PAGE_NUMER}="${elementPageNumber}"].js-pagination-number`,
          );
          element2.classList.add(this.CLASSES.IS_ACTIVE);
        }
      });

      this.$emit('page-changed', elementPageNumber);
      this.currentPageNumber = elementPageNumber;
    },
  },
};
</script>

<style lang="scss" scoped>
@import '../../assets/styles/abstracts/index';

.c-pagination {
  $padding: px-to-rem(16px);

  @include list-unstyled($margin-bottom: 0);

  display: flex;
  align-items: center;
  margin-top: 0;
  background-color: $color-aqua-haze;
  border-radius: px-to-rem(24px);
  font-size: $g-font-size-small;

  &__button {
    display: flex;
    height: px-to-rem(24px);
    align-items: center;
    justify-content: center;
    padding: $padding ($padding / 1.6);
    cursor: pointer;
    line-height: 1;

    &:hover,
    &.is-active {
      color: $color-primary;
    }

    &.is-active {
      color: $color-primary;
      font-weight: 500;
    }
  }

  &__item {
    &:first-of-type .c-pagination__button {
      padding-left: $padding;
    }

    &:last-of-type .c-pagination__button {
      padding-right: $padding;
    }
  }
}
</style>
