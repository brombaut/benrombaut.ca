<template>
  <section id="bookshelf">
    <SectionHeader
      title="Ben's Bookshelf"
      icon="book-open"
      subtext="Books I've read and am currently reading."/>
    <div v-if="loadingBookshelf" class="section-body">
      <div class="loader-wrapper">
        <div class="loader"></div>
      </div>
    </div>
    <div v-else class="section-body">
      <div class="book-group">
        <h2 class='book-group-header'>Currently Reading</h2>
        <div class="books">
          <BookCard v-for="book in currentlyReadingBooks" :key="book.title" :book="book" />
        </div>
      </div>
      <div v-for="yearBookGroup in readBooksByYear" :key="yearBookGroup.year" class="book-group">
        <h2 class='book-group-header'>{{ yearBookGroup.year }}</h2>
        <div class="books">
          <BookCard v-for="book in yearBookGroup.books" :key="book.title" :book="book" />
        </div>
      </div>
    </div>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Book } from "@brombaut/types";
import SectionHeader from "../shared/SectionHeader.vue";
import BookCard from "./BookCard.vue";
import CachedF3Bookshelf from "./CachedF3Bookshelf";

type YearBooksPair = { year: number; books: Book[] };

@Component({
  components: {
    BookCard,
    SectionHeader,
  },
})
export default class BookshelfSection extends Vue {
  booksLoading = true;

  constructor() {
    super();
    this.loadingBookshelf = true;
    this.initBookshelf();
  }

  async initBookshelf() {
    await CachedF3Bookshelf.init();
    this.booksLoading = false;
  }

  set loadingBookshelf(val: boolean) {
    this.booksLoading = val;
  }

  get loadingBookshelf(): boolean {
    return this.booksLoading;
  }

  get readBooksByYear(): YearBooksPair[] {
    if (this.loadingBookshelf) {
      return [];
    }
    const bookGroups: YearBooksPair[] = [];
    const keyVals = CachedF3Bookshelf.readBooksGroupedByYear();
    Object.entries(keyVals).forEach(keyVal => {
      bookGroups.push({ year: Number(keyVal[0]), books: keyVal[1] });
    });
    return bookGroups.sort((a: YearBooksPair, b: YearBooksPair) => {
      return b.year - a.year;
    });
  }

  get currentlyReadingBooks(): Book[] {
    if (this.loadingBookshelf) {
      return [];
    }
    return CachedF3Bookshelf.readingBooks();
  }
}
</script>

<style lang="scss">
#bookshelf {
  display: flex;
  flex-direction: column;

  .section-body {
    display: flex;
    flex-direction: column;
    align-items: flex-start;

    .loader-wrapper {
      width: 100%;
      display: flex;
      justify-content: center;

      .loader {
        border: 16px solid $primaryDarkest;
        border-top: 16px solid $primary;
        border-bottom: 16px solid $primary;
        border-radius: 50%;
        width: 120px;
        height: 120px;
        animation: spin 1s linear infinite;
      }

      @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
      }
    }

    .book-group {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      width: 100%;
      margin-bottom: 16px;

      .book-group-header {
        // margin: 12px 0;
        text-decoration: underline;
      }

      .books {
        display: flex;
        flex-direction: row;
        align-items: flex-start;
        justify-content: flex-start;
        flex-wrap: wrap;
      }
    }
  }
}
</style>
