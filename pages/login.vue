<template>
	<div class="login-container">
		<v-card min-width="700">
			<v-card-title><p class="">Login</p></v-card-title>
			<v-card-text>
				<v-form @submit.prevent="login">
					<v-text-field
						outlined
						label="Username"
						dense
						v-model="username"
					></v-text-field>
					<v-text-field
						v-model="password"
						outlined
						label="Password"
						type="password"
						dense
					></v-text-field>
					<v-btn color="primary" type="submit">Login</v-btn>
				</v-form>
			</v-card-text>
		</v-card>
	</div>
</template>

<script>
import Cookies from 'js-cookie'

export default {
	data() {
		return {
			username: null,
			password: null,
		}
	},
	methods: {
		login() {
			this.$axios({
				method: 'POST',
				data: {
					username: this.username,
					password: this.password,
				},
				url: '/user/login',
			})
				.then(({ data }) => {
					Cookies.set('user_token', data.data.token, { expires: 30 })

					this.$axios.defaults.headers.common[
						'Authorization'
					] = `Bearer ${data.data.token}`

					this.$router.push('/')
				})
				.catch(err => {
					alert(err.response.data.message)
				})
		},
	},
}
</script>

<style scoped lang="scss">
.login-container {
	height: 100vh;
	width: 100%;
	display: flex;
	align-items: center;
	justify-content: center;
}
</style>
