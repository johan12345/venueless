<template lang="pug">
.c-upload-url-input
	bunt-input(:value="value", :label="label", :name="name", :validation="validation", @input="update($event)")
	bunt-progress-circular(v-if="uploading", size="small")
	.file-selector(v-else)
		bunt-icon-button upload
		input(type="file", @change="upload", ref="fileInput")

</template>
<script>
import config from 'config'

export default {
	props: {
		value: String,
		label: String,
		name: String,
		validation: Object,
	},
	data () {
		return {
			uploading: false,
		}
	},
	methods: {
		update (val) {
			this.$emit('input', val)
		},
		upload () {
			const data = new FormData()
			var file = this.$refs.fileInput.files[0]
			data.append('file', file)

			const headers = new Headers()
			if (this.$store.state.token) {
				headers.append('Authorization', `Bearer ${this.$store.state.token}`)
			} else if (this.$store.state.clientId) {
				headers.append('Authorization', `Client ${this.$store.state.clientId}`)
			}
			this.uploading = true
			fetch(config.api.upload, {
				method: 'POST',
				headers: headers,
				body: data
			}).then(response => response.json()).then(data => {
				if (data.error) {
					alert(`Upload error: ${data.error}`) // Proper user-friendly messages
					this.$emit('input', '')
				} else {
					this.$emit('input', data.url)
				}
				this.uploading = false
			}).catch(error => {
				// TODO: better error handling
				console.log(error)
				alert(`error: ${error}`)
				this.uploading = false
			})
		}
	},
}
</script>
<style lang="stylus">
	.c-upload-url-input
		display: flex
		align-items: center
		.bunt-progress-circular.small
			margin: 0 6px
		.bunt-input
			flex-grow: 1

		.file-selector
			flex-grow: 0
			position: relative

			input
				opacity: 0
				cursor: pointer
				position: absolute
				width: 100%
				height: 100%
				top: 0
				left: 0
</style>
