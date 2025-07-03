

<template>
<li  class="postItem">
    <h3 class="postTitle">{{ post.title }}</h3>
    <p class="postText" >
      {{ displayBody }}
      <a class="postControlText" v-if="shouldShowReadMore" @click="toggleReadMore" >
        {{ isExpanded ? 'collapse' : 'show more' }}
      </a>
    </p>
    <div class="authorBox" >
      <span >Autor: {{ authorName }}</span>
      <button class="deleteBtn"  @click="emitDelatePost">Delete post</button>
    </div>
  </li>
</template>

<script>
export default {
  
  name: 'PostItem',
  
  props: {
    post: {
      type: Object, 
      required: true, 
    },
    maxLength: { 
      type: Number,
      default: 50, 
    }
  },
  data() {
    return {
      isExpanded: false, 
      author: null,      
      loadingAuthor: false, 
      authorError: null,   
    };
  },
  computed: {
   
    displayBody() {
     
      if (this.isExpanded || this.post.body.length <= this.maxLength) {
        return this.post.body;
      }
     
      return this.post.body.substring(0, this.maxLength) + '...';
    },
    
    shouldShowReadMore() {
     
      return this.post.body.length > this.maxLength;
    },
    
    authorName() {
      if (this.loadingAuthor) {
        return 'Loading author...';
      }
      if (this.authorError) {
        return 'Error loading author.';
      }
      
      return this.author ? this.author.name : 'Unknown author';
    }
  },
  methods: {
   
    toggleReadMore() {
      this.isExpanded = !this.isExpanded;
    },
    
    async fetchAuthor() {
      this.loadingAuthor = true;
      this.authorError = null;
      try {
        
        const response = await fetch(`https://jsonplaceholder.typicode.com/users/${this.post.userId}`);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        this.author = data; 
      } catch (e) {
        this.authorError = 'Failed to get author: ' + e.message;
        console.error('Author download error:', e);
      } finally {
        this.loadingAuthor = false;
      }
    },
    
    emitDelatePost() {
     
      
        this.$emit('delete-post', this.post.id); 
      
    }
  },
 
  created() {
    this.fetchAuthor();
  },
};
</script>

