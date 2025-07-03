<template>
	<div>
		<header>
			<h1>List of posts:</h1>
		</header>

		<main class='wrapper'>
			<div v-if="loading">...loading posts</div>
			<div v-else-if="error">An error occurred: {{ error }}</div>
			<ul class='postsList' v-else>
				<PostItem
					v-for="post in paginatedPosts"
					:key="post.id"
					:post="post"
					:maxLength="50"
					@delete-post="handleDeletePost" />
			</ul>

			<div class="pagination">
				<button @click="changePage(currentPage - 1)" :disabled="currentPage === 1" class="paginationButton">
					Previous
				</button>
				<span class="pageInfo">Page {{ currentPage }} of {{ totalPages }}</span>
				<button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages" class="paginationButton">
					Next
				</button>
			</div>
		</main>
	</div>
</template>

<script>
import PostItem from './components/PostItem.vue'

export default {
	name: 'App',
	components: {
		PostItem,
	},
	data() {
		return {
			posts: [],
			loading: false,
			error: null,

			currentPage: 1,
			postsPerPage: 10,
			totalPosts: 0,
		}
	},
	computed: {
		paginatedPosts() {
			const startIndex = (this.currentPage - 1) * this.postsPerPage
			const endIndex = startIndex + this.postsPerPage
			return this.posts.slice(startIndex, endIndex)
		},

		totalPages() {
			return Math.ceil(this.posts.length / this.postsPerPage)
		},
	},
	methods: {
		async fetchPosts() {
			this.loading = true
			this.error = null

			try {
				const response = await fetch('https://jsonplaceholder.typicode.com/posts')
				if (!response.ok) {
					throw new Error(`Error HTTP! Status: ${response.status}`)
				}
				const data = await response.json()
				this.posts = data
				this.totalPosts = data.length
			} catch (e) {
				this.error = 'Failed to fetch posts: ' + e.message
				console.error('Error downloading posts:', e)
			} finally {
				this.loading = false
			}
		},

		changePage(page) {
			if (page >= 1 && page <= this.totalPages) {
				this.currentPage = page
			}
		},

		handleDeletePost(postId) {
			this.posts = this.posts.filter(post => post.id !== postId)

			if (this.paginatedPosts.length === 0 && this.currentPage > 1) {
				this.currentPage--
			}
		},
	},

	created() {
		this.fetchPosts()
	},
}
</script>
