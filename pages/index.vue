<template>
	<div>
		<div v-if="user">
			<v-card class="navbar-container">
				<v-card-title class="px-10 d-flex justify-space-between">
					<div>
						Welcome &nbsp;<span class="primary--text font-weight-bold"
							>{{ user.username }}
						</span>
					</div>
					<div class="search-container">
						<v-text-field
							outlined
							label="Search a user..."
							dense
							hide-details
							v-model="search"
						></v-text-field>
					</div>
				</v-card-title>
			</v-card>
			<div class="d-flex">
				<v-card class="sidebar-container">
					<v-list class="pa-0">
						<v-list-item-group v-model="selectedItem" color="primary">
							<v-list-item
								v-for="(item, i) in chats"
								:key="i"
								@click="joinRoom(item._id)"
							>
								<v-list-item-icon>
									<v-icon>{{
										item.type === 'private'
											? 'mdi-account'
											: 'mdi-account-group'
									}}</v-icon>
								</v-list-item-icon>
								<v-list-item-content>
									<v-list-item-title
										v-text="item.name"
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
					<div v-else class="chat-container">
						<!--            MESSAGES ARE HEREEEEEE-->
						<div class="msg-area-container"></div>
						<v-form @submit.prevent="sendMsg" class="msg-box-container">
							<v-text-field
								outlined
								label="Type a message..."
								dense
								class="msg-box"
								hide-details
								v-model="message"
							></v-text-field>
							<v-btn
								color="primary"
								class="send-btn"
								icon
								type="submit"
								x-large
							>
								<v-icon>mdi-send</v-icon>
							</v-btn>
						</v-form>
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
import { io } from 'socket.io-client'

export default {
	data() {
		return {
			user: null,
			selectedItem: null,
			search: null,
			chats: [],
			socket: null,
			message: null,
		}
	},
	mounted() {
		this.socket = io('ws://localhost:3001')

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
			.then(({ data }) => {
				this.user = data.data.user

				this.$axios({
					method: 'GET',
					url: '/user/chats',
				}).then(({ data }) => {
					// get chats
					this.chats = data.data.chats
				})
				// .catch(() => this.$router.push('/login'))
			})
			.catch(() => this.$router.push('/login'))

		this.socket.on('msg', data => console.log(data))
	},
	methods: {
		joinRoom(roomId) {
			this.socket.emit('joinRoom', {
				_id: this.user._id,
				room: roomId,
			})
		},
		sendMsg() {
			const room = this.chats[this.selectedItem]._id

			console.log(this.message + 'to: ' + room)

			// do socket things then

			// clear message
			this.message = null
		},
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
	padding-top: 75px;
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
	padding-top: 75px;
}

.start-msg {
	display: flex;
	align-items: center;
	justify-content: center;
	height: 100%;
}

.chat-container {
	height: 100%;
	display: flex;
	flex-direction: column;

	.msg-area-container {
		flex: 1;
		padding: 10px;
	}

	.msg-box-container {
		display: flex;
		padding: 10px;
		align-items: center;
	}
}
</style>
