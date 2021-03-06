<template>
  <div class="comments">
    <div class="gel-wrap">
      <div class="gel-layout gel-layout--center">
        <div class="gel-layout__item gel-10/12@m gel-8/12@l">
          <div class="gel-layout">

            <!-- Title -->
            <div class="gel-layout__item">
              <div class="comments__title gel-paragon-bold">
                Two lines max: call to action, title or question?
              </div>
            </div>

            <!-- Add a Comment -->
            <div class="gel-layout__item">
              <bbc-submit-comment
                :display-name="displayName"
                :placeholder-text="'Comment as ' + displayName"
                @comment-submitted="submitComment"
                cta-text="Add comment">
              </bbc-submit-comment>
            </div>

            <!-- Num. Comments -->
            <div class="gel-layout__item">
              <div class="comments__number gel-pica">
                <span>{{ comments.length }}</span> Comments
              </div>
            </div>

            <!-- Sorting/Filtering -->
            <div class="gel-layout__item comments__filter-order">
              <select name="showCommentType">
                <option value="all" selected="">Show everything</option>
              </select>

              <select name="orderCommentsBy">
                <option value="latest" selected="">Latest first</option>
              </select>
            </div>

            <!-- Comments! -->
            <div class="gel-layout__item">
              <transition-group name="new-comment" tag="div">
                <bbc-comment
                  v-for="comment in
                    getOrderedFilteredAndLimitComments(comments, numVisibleComments)"
                  :key="comment.id"

                  :display-name="comment.displayName"
                  :comment-text="comment.commentText"
                  :timestamp="comment.timestamp"
                  :num-up-votes="comment.numUpVotes"
                  :num-down-votes="comment.numDownVotes"
                  :replies="comment.replies"
                ></bbc-comment>
              </transition-group>
            </div>

            <button
              class="comments__show-more"
              v-show="isCommentsLimited"
              @click="showMoreComments()">
                More Comments
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import commentsData from '../comments';
import BbcSubmitComment from './BbcSubmitComment';
import BbcComment from './BbcComment';

const filters = {
  all(comments) {
    return comments;
  },
};

const orders = {
  oldest(comments) {
    return comments;
  },

  newest(comments) {
    return comments.slice(0).reverse();
  },
};

export default {
  props: {
    displayName: String,
  },

  components: { BbcSubmitComment, BbcComment },

  data() {
    return {
      activeFilter: 'all',
      activeOrder: 'newest',
      numVisibleComments: 3,
      isCommentsLimited: false,

      // DATA SCHEMA
      // -----------
      // id: this.nextCommentId += 1, // Increment `nextCommentId` for next comment.
      // displayName: this.displayName,
      // commentText: this.commentText,
      // timestamp: new Date(),
      // numUpVotes: 0,
      // numDownVotes: 0,
      // replies: [],

      comments: commentsData,
    };
  },

  methods: {
    submitComment(commentText) {
      const nextCommentId = this.comments.length + 1;
      this.comments.push({
        id: nextCommentId, // Increment `nextCommentId` for next reply.
        displayName: this.displayName,
        commentText,
        timestamp: new Date(),
        numUpVotes: 0,
        numDownVotes: 0,
        replies: [],
      });
    },

    getOrderedAndFilteredComments(rawComments) {
      // 1. Order
      // 2. Filter
      let comments = orders[this.activeOrder](rawComments);
      comments = filters[this.activeFilter](comments);

      // Watch/set this elsewhere?
      if (comments.length > this.numVisibleComments) {
        this.isCommentsLimited = true;
      } else {
        this.isCommentsLimited = false;
      }

      return comments;
    },

    getOrderedFilteredAndLimitComments(rawComments, numComments) {
      if (typeof numComments !== 'number') {
        throw Error('getOrderedFilteredAndLimitComments should be passed a Number as the second argument.');
      }
      const comments = this.getOrderedAndFilteredComments(rawComments);
      return comments.slice(0, numComments);
    },

    showMoreComments() {
      const numTotalComments = this.getOrderedAndFilteredComments(this.comments).length;

      if (numTotalComments > this.numVisibleComments) {
        this.numVisibleComments += 10;
      }
    },
  },
};
</script>

<style lang="scss" scoped="">
  .comments {
    background-color: #CCC;
    padding-top: 64px;
    padding-bottom: 28px;
    position: relative;
  }
    .comments__title {
      margin-bottom: 24px;

      > span {
        font-weight: bold;
      }
    }

    .comments__filter-order {
      padding-top: 12px;
      padding-bottom: 20px;
    }

    .comments__number {
      margin-top: 24px;

      > span {
        font-weight: bold;
      }
    }

    .comments__show-more {
      background-color: #FFF;
      font-weight: bold;
      padding: 12px 32px;
      position: absolute;
        bottom: 0; left: 50%;
      transform: translate(-50%, 50%);
      white-space: nowrap;
    }

  // Animation
  //
  .new-comment {}
  .new-comment-leave-to {
    transform: translateY(32px);
  }

  // Enter
  .new-comment-enter {
    opacity: 0;
    transform: translateY(-32px);
  }
  .new-comment-enter-active {
    transition: all 350ms ease-out;
  }
  .new-comment-enter-to {}

  // Leave
  .new-comment-leave {}
  .new-comment-leave-active {}
  .new-comment-leave-to {}
</style>
