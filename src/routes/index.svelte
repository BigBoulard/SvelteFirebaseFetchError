<script context="module">
	// import serviceAccount from '../private/my-firebase-private-key.json'
	import { google } from 'googleapis'

	let firebaseAccessToken

	const serviceAccount = {
		'type': 'service_account',
		'project_id': 'my-test-project-xxxxxx',
		'private_key_id': '0d33XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
		'private_key':
			'-----BEGIN PRIVATE KEY-----\nXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX-----END PRIVATE KEY-----\n',
		'client_email':
			'firebase-adminsdk-xxxxx@my-test-project-xxxxxx.iam.gserviceaccount.com',
		'client_id': '9999999999999999999999',
		'auth_uri': 'https://accounts.google.com/o/oauth2/auth',
		'token_uri': 'https://oauth2.googleapis.com/token',
		'auth_provider_x509_cert_url': 'https://www.googleapis.com/oauth2/v1/certs',
		'client_x509_cert_url':
			'https://www.googleapis.com/robot/v1/metadata/x509/firebase-adminsdk-xxxxxx%40my-test-project-xxxxxx.iam.gserviceaccount.com',
	}

	async function firebaseAuth() {
		let scopes = [
			'https://www.googleapis.com/auth/userinfo.email',
			'https://www.googleapis.com/auth/firebase.database',
		]

		let jwtClient = new google.auth.JWT(
			serviceAccount.client_email,
			null,
			serviceAccount.private_key,
			scopes
		)

		return new Promise((resolve, reject) =>
			jwtClient.authorize((error, tokens) => {
				if (error) {
					console.log('Error making request to generate access token:', error)
					reject(error)
				} else if (tokens.access_token === null) {
					console.log(
						'Provided service account does not have permission to generate access tokens'
					)
					reject(error)
				} else {
					firebaseAccessToken = tokens.access_token
					resolve(tokens.access_token)
				}
			})
		)
	}

	// export async function load({ page, fetch, session, context }) {
	export async function load({ fetch }) {
		await firebaseAuth()
		// console.log(`firebaseAccessToken: ${firebaseAccessToken}`)
		const res = await fetch(
			'https://YOUR_DATABASE.firebasedatabase.app/meetups.json',
			{
				headers: { 'Authorization': `Bearer ${firebaseAccessToken}` },
			}
		)
		const data = await res.json()

		if (!res.ok) {
			return {
				status: res.status,
				error: new Error('Could not fetch data'),
			}
		}

		return { props: { myData: data } }
	}
</script>

<script>
	import { onMount } from 'svelte'
	export let myData

	onMount(() => {
		console.log(myData)
	})
</script>
