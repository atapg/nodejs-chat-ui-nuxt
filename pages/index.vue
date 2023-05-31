<template>
	<div>
		<div v-if="user">
			<v-card class="navbar-container">
				<v-card-title>
					Welcome &nbsp;<span class="primary--text font-weight-bold"
						>{{ user.username }}
					</span>
				</v-card-title>
			</v-card>
			<div class="d-flex">
				<v-card class="sidebar-container">
					<v-list class="pa-0">
						<v-list-item-group v-model="selectedItem" color="primary">
							<v-list-item v-for="(item, i) in items" :key="i">
								<v-list-item-icon>
									<v-icon v-text="item.icon"></v-icon>
								</v-list-item-icon>
								<v-list-item-content>
									<v-list-item-title
										v-text="item.text"
									></v-list-item-title>
								</v-list-item-content>
							</v-list-item>
						</v-list-item-group>
					</v-list>
				</v-card>
				<div class="chat-area-container">
					<div v-if="selectedItem === null" class="start-msg">
						<p class="headline">Select a chat to start messaging!</p>
					</div>
				</div>
			</div>
		</div>
		<div class="loader" v-else>
			<v-progress-circular
				indeterminate
				color="primary"
				size="100"
			></v-progress-circular>
		</div>
	</div>
</template>

<script>
import Cookies from 'js-cookie'

export default {
	data() {
		return {
			user: null,
			selectedItem: null,
			items: [
				{ text: 'Real-Time', icon: 'mdi-clock' },
				{ text: 'Real-Time', icon: 'mdi-clock' },
				{ text: 'Real-Time', icon: 'mdi-clock' },
			],
		}
	},
	mounted() {
		const token = Cookies.get('user_token')

		if (token) {
			this.$axios.defaults.headers.common[
				'Authorization'
			] = `Bearer ${token}`
		} else {
			this.$router.push('/login')
		}

		this.$axios({
			method: 'GET',
			url: '/user',
		})
			.then(({ data }) => (this.user = data.data.user))
			.catch(() => this.$router.push('/login'))
	},
}
</script>

<style scoped lang="scss">
.home-container {
	height: 100vh;
	width: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
}

.loader {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translateX(-50%) translateY(-50%);
}

.navbar-container {
	z-index: 2;
	position: absolute;
	top: 0;
	left: 0;
	right: 0;
}

.sidebar-container {
	padding-top: 65px;
	z-index: 1;
	position: relative;
	max-width: 300px;
	width: 300px;
	height: 100vh;
}

.chat-area-container {
	width: 100%;
	height: 100vh;
	padding: 5px;
	padding-top: 65px;
}

.start-msg {
	display: flex;
	align-items: center;
	justify-content: center;
	height: 100%;
}
</style>
