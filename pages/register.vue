<template>
	<div class="login-container">
		<v-card min-width="700">
			<v-card-title><p class="">Register</p></v-card-title>
			<v-card-text>
				<v-form @submit.prevent="register">
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
					<v-btn
						color="primary"
						type="submit"
						:disabled="!password || !username"
						>Register</v-btn
					>
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
		register() {
			this.$axios({
				method: 'POST',
				data: {
					username: this.username,
					password: this.password,
				},
				url: '/user/register',
			})
				.then(({ data }) => {
					this.$router.push('/login')
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
