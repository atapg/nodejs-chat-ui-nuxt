<template>
	<div>
		<div v-if="user">
			<v-card class="navbar-container">
				<v-card-title class="px-10 d-flex justify-space-between">
					<div>
						&nbsp;<span class="primary--text font-weight-bold">
							<v-menu offset-y>
								<template v-slot:activator="{ on, attrs }">
									<span class="welcome-text"> Welcome</span>
									<v-btn
										color="primary"
										dark
										v-bind="attrs"
										v-on="on"
										text
									>
										{{ user.username }}
									</v-btn>
								</template>
								<v-list>
									<v-list-item link @click="createChatDialog = true">
										<v-list-item-icon>
											<v-icon>mdi-plus</v-icon>
										</v-list-item-icon>
										<v-list-item-title>
											Create New Chat
										</v-list-item-title>
									</v-list-item>
									<v-list-item link @click="logout">
										<v-list-item-icon>
											<v-icon>mdi-location-exit</v-icon>
										</v-list-item-icon>
										<v-list-item-title> Logout </v-list-item-title>
									</v-list-item>
								</v-list>
							</v-menu>
						</span>
					</div>
					<div class="search-container">
						<!--						<v-text-field-->
						<!--							outlined-->
						<!--							label="Search a user..."-->
						<!--							dense-->
						<!--							hide-details-->
						<!--							v-model="search"-->
						<!--						></v-text-field>-->
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
						<div class="go-bottom"></div>
						<!--MESSAGES ARE HEREEEEEE-->

						<div class="chats-area-container" ref="chatScrollableArea">
							<div
								class="align-center text-center mb-10"
								v-if="!fetchingNewChats"
							>
								<v-btn
									color="primary"
									text
									small
									class="align-center"
									@click="fetchNewChats"
									>Load More...</v-btn
								>
							</div>
							<div
								:class="`msg-area-container ${
									msg.from._id === user._id ? 'pb-0 pt-1' : ''
								} ${
									index > 0
										? msg.from._id ===
										  currentChats[index - 1].from._id
											? 'pb-0 pt-1'
											: ''
										: null
								}  `"
								v-for="(msg, index) in currentChats"
								:key="index"
							>
								<div
									:class="`msg-container d-flex align-center  ${
										msg.from._id === user._id ? 'show-on-right' : ''
									}`"
								>
									<div v-if="msg.from._id !== user._id">
										<v-btn
											:color="msg.from.avatarColor"
											icon
											:class="`mr-2`"
											v-if="
												index > 0
													? msg.from._id ===
													  currentChats[index - 1].from._id
														? ''
														: true
													: true
											"
										>
											<v-icon>mdi-account</v-icon>
										</v-btn>
									</div>

									<div
										:class="`mb-0 msg-box ${
											msg.from._id === user._id ? 'primary' : ''
										} ${
											index > 0
												? msg.from._id ===
												  currentChats[index - 1].from._id
													? 'ml-11'
													: true
												: null
										}`"
									>
										<div>
											<div v-if="msg.from._id !== user._id">
												<p
													v-if="
														index > 0
															? msg.from._id ===
															  currentChats[index - 1].from._id
																? ''
																: true
															: true
													"
													class="primary--text font-weight-bold username-text mb-3"
												>
													{{ msg.from.username }}
												</p>
											</div>
											<p>{{ msg.content }}</p>
										</div>
										<span class="msg-time grey--text">{{
											new Date(msg.createdAt).toLocaleTimeString()
										}}</span>
									</div>
								</div>
							</div>
						</div>

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
								:disabled="!message"
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
		<v-dialog v-model="createChatDialog" width="500">
			<v-card>
				<v-card-title>Create new chat</v-card-title>
				<v-card-text>
					<v-form @submit.prevent="createChatHandler">
						<v-text-field
							outlined
							label="Chat name"
							dense
							v-model="createChatForm.name"
						></v-text-field>
						<!--						<v-text-field-->
						<!--							outlined-->
						<!--							label="Search members..."-->
						<!--							dense-->
						<!--						></v-text-field>-->
						<span>Members</span>
						<div class="members-area-container mt-2">
							<v-list class="pa-0">
								<div v-for="member in members" :key="member._id">
									<v-list-item>
										<v-list-item-icon>
											<v-icon :color="member.avatarColor">
												mdi-account
											</v-icon>
										</v-list-item-icon>
										<v-list-item-content>
											<v-list-item-title>
												{{ member.username }}
											</v-list-item-title>
										</v-list-item-content>
										<v-list-item-action>
											<v-btn icon @click="addMember(member._id)">
												<v-icon
													color="grey lighten-1"
													v-if="
														createChatForm.members.indexOf(
															member._id,
														) < 0
													"
												>
													mdi-plus
												</v-icon>
												<v-icon v-else> mdi-close </v-icon>
											</v-btn>
										</v-list-item-action>
									</v-list-item>
									<v-divider></v-divider>
								</div>
							</v-list>
						</div>
						<v-btn
							color="primary"
							type="submit"
							:loading="createChatLoading"
							:disabled="
								!createChatForm.name ||
								createChatForm.members.length <= 0
							"
							>Create Chat</v-btn
						>
					</v-form>
				</v-card-text>
			</v-card>
		</v-dialog>
	</div>
</template>

<script>
import Cookies from 'js-cookie'
import { io } from 'socket.io-client'

export default {
	components: {},
	data() {
		return {
			user: null,
			selectedItem: null,
			search: null,
			chats: [],
			socket: null,
			message: null,
			messages: [],
			currentChats: [],
			chatLoading: false,
			hasReachedToTop: false,
			fetchingNewChats: false,
			page: 1,
			createChatDialog: false,
			members: [],
			createChatForm: {
				members: [],
				name: null,
			},
			createChatLoading: false,
		}
	},
	mounted() {
		//TODO add header for socket
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
				this.socket = io('ws://localhost:3001', {
					query: `_id=${this.user._id}`,
				})

				this.socket.emit('addSocket', { _id: this.user._id })

				this.socket.on('msg', data => {
					this.currentChats.push(data)

					// Niceeeee wait for v-for to finish then run this func!!!!!!
					this.$nextTick(() => {
						this.$refs.chatScrollableArea.scrollTop =
							this.$refs.chatScrollableArea.scrollHeight
					})
				})

				this.socket.on('chat', () => {
					this.getChats()
				})

				this.getChats()
				// .catch(() => this.$router.push('/login'))
			})
			.catch(() => this.$router.push('/login'))

		this.$axios({
			method: 'GET',
			url: '/chat/members',
		}).then(({ data }) => {
			this.members = data.data
		})
	},
	beforeDestroy() {
		this.socket.disconnect()
	},
	methods: {
		fetchNewChats() {
			this.fetchingNewChats = true
			this.getChatMessages(
				this.chats[this.selectedItem]._id,
				this.messages.filter(
					m => m.roomId === this.chats[this.selectedItem]._id,
				)[0].page,
			)
		},
		getChats() {
			this.$axios({
				method: 'GET',
				url: '/user/chats',
			}).then(({ data }) => {
				// get chats
				this.chats = data.data.chats
			})
		},
		joinRoom(roomId) {
			this.socket.emit('joinRoom', {
				_id: this.user._id,
				room: roomId,
			})

			// this.getChatMessages(roomId, 1)
		},
		sendMsg() {
			// do socket things then
			this.socket.emit('msg', {
				room: this.chats[this.selectedItem]._id,
				msg: this.message,
				_id: this.user._id,
			})

			// clear message
			this.message = null
		},
		getChatMessages(roomId, page = 1) {
			this.$axios({
				method: 'GET',
				url: `/chat/messages/${roomId}?page=${page}`,
			})
				.then(({ data }) => {
					// make condition for first fetch

					if (data.data.length === 0 || data.data.length < 20) {
						this.fetchingNewChats = true
					} else {
						this.fetchingNewChats = false
					}

					if (page === 1) {
						this.messages.push({
							roomId,
							messages: data.data.reverse(),
							firstFetch: true,
							page: 1,
						})
					} else {
						data.data.forEach(m => {
							this.messages
								.filter(m => m.roomId === roomId)[0]
								.messages.unshift(m)
						})
					}

					this.currentChats = this.messages.filter(
						c => c.roomId === this.chats[this.selectedItem]._id,
					)[0].messages

					// Niceeeee wait for v-for to finish then run this func!!!!!!
					if (page === 1) {
						this.$nextTick(() => {
							this.$refs.chatScrollableArea.scrollTop =
								this.$refs.chatScrollableArea.scrollHeight
						})
					}

					this.messages.filter(m => m.roomId === roomId)[0].page =
						this.messages.filter(m => m.roomId === roomId)[0].page + 1
				})
				.catch(e => {})
		},
		logout() {
			this.user = null
			Cookies.remove('user_token')
			this.$router.push('/login')
		},
		addMember(memberId) {
			const memberAlreadyAdded = this.createChatForm.members.filter(
				member => member === memberId,
			)

			if (memberAlreadyAdded.length > 0) {
				// remove
				this.createChatForm.members.splice(
					this.createChatForm.members.indexOf(memberId),
					1,
				)
			} else {
				this.createChatForm.members.push(memberId)
			}
		},
		createChatHandler() {
			this.createChatLoading = true

			this.$axios({
				method: 'POST',
				url: '/chat',
				data: {
					members: this.createChatForm.members,
					name: this.createChatForm.name,
				},
			})
				.then(({ data }) => {
					this.getChats()
					this.createChatForm.members = []
					this.createChatForm.name = null
				})
				.finally(() => {
					this.createChatDialog = false
					this.createChatLoading = false
				})
		},
	},
	watch: {
		selectedItem() {
			if (this.selectedItem !== undefined) {
				const roomId = this.chats[this.selectedItem]._id

				if (this.messages.filter(c => c.roomId === roomId).length <= 0) {
					this.getChatMessages(roomId, 1)
				} else {
					if (this.messages.filter(c => c.roomId === roomId)[0]) {
						this.currentChats = this.messages.filter(
							c => c.roomId === this.chats[this.selectedItem]._id,
						)[0].messages
					} else {
						this.getChatMessages(roomId, 1)
					}
				}
			}
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
	overflow: auto;
	-ms-overflow-style: none; /* Internet Explorer 10+ */
	scrollbar-width: none; /* Firefox */
}

.sidebar-container::-webkit-scrollbar {
	display: none; /* Safari and Chrome */
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
	justify-content: flex-end;

	.msg-area-container {
		//flex: 1;
		padding-top: 25px;
	}

	.msg-box-container {
		display: flex;
		padding: 10px;
		align-items: center;
	}
}

.chats-area-container {
	//height: 100%;
	overflow: auto;
	overflow-x: hidden;
	padding: 20px;
	padding-bottom: 0;
	padding-top: 0;
}

.go-bottom {
	//flex: 1;
}

.msg-area-container {
}

.msg-container {
	.msg-box {
		padding: 7px;
		padding-left: 20px;
		padding-right: 20px;
		border-radius: 5px;
		background-color: $grey;
		max-width: 400px;
		min-width: 200px;
		display: flex;
		align-items: flex-end;
		justify-content: space-between;

		span {
			margin-left: 10px;
			white-space: nowrap;
		}
	}

	.msg-time {
		font-size: 10px;
		text-align: right;
		display: block;
	}
}
.show-on-right {
	justify-content: flex-end;
}

.username-text {
	cursor: pointer;
	user-select: none;
	margin-bottom: 10px;
}

.welcome-text {
	color: rgba(255, 255, 255, 1) !important;
	font-size: 1rem;
	margin-right: 10px;
}

.members-area-container {
	height: 200px;
	margin-bottom: 30px;
	border-radius: 5px;
	border: 1px rgba(255, 255, 255, 0.24) solid;
	padding: 10px;
	overflow: auto;
}
</style>
