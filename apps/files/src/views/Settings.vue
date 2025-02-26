<!--
  - @copyright Copyright (c) 2019 Gary Kim <gary@garykim.dev>
  -
  - @author Gary Kim <gary@garykim.dev>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->
<template>
	<NcAppSettingsDialog :open="open"
		:show-navigation="true"
		:title="t('files', 'Files settings')"
		@update:open="onClose">
		<!-- Settings API-->
		<NcAppSettingsSection id="settings" :title="t('files', 'Files settings')">
			<NcCheckboxRadioSwitch :checked.sync="show_hidden"
				@update:checked="setConfig('show_hidden', $event)">
				{{ t('files', 'Show hidden files') }}
			</NcCheckboxRadioSwitch>
			<NcCheckboxRadioSwitch :checked.sync="crop_image_previews"
				@update:checked="setConfig('crop_image_previews', $event)">
				{{ t('files', 'Crop image previews') }}
			</NcCheckboxRadioSwitch>
		</NcAppSettingsSection>

		<!-- Settings API-->
		<NcAppSettingsSection v-if="settings.length !== 0"
			id="more-settings"
			:title="t('files', 'Additional settings')">
			<template v-for="setting in settings">
				<Setting :key="setting.name" :el="setting.el" />
			</template>
		</NcAppSettingsSection>

		<!-- Webdav URL-->
		<NcAppSettingsSection id="webdav" :title="t('files', 'Webdav')">
			<NcInputField id="webdav-url-input"
				:show-trailing-button="true"
				:success="webdavUrlCopied"
				:trailing-button-label="t('files', 'Copy to clipboard')"
				:value="webdavUrl"
				readonly="readonly"
				type="url"
				@focus="$event.target.select()"
				@trailing-button-click="copyCloudId">
				<template #trailing-button-icon>
					<Clipboard :size="20" />
				</template>
			</NcInputField>
			<em>
				<a :href="webdavDocs" target="_blank" rel="noreferrer noopener">
					{{ t('files', 'Use this address to access your Files via WebDAV') }} ↗
				</a>
			</em>
		</NcAppSettingsSection>
	</NcAppSettingsDialog>
</template>

<script>
import NcAppSettingsDialog from '@nextcloud/vue/dist/Components/NcAppSettingsDialog.js'
import NcAppSettingsSection from '@nextcloud/vue/dist/Components/NcAppSettingsSection.js'
import NcCheckboxRadioSwitch from '@nextcloud/vue/dist/Components/NcCheckboxRadioSwitch.js'
import Clipboard from 'vue-material-design-icons/Clipboard.vue'
import NcInputField from '@nextcloud/vue/dist/Components/NcInputField'
import Setting from '../components/Setting.vue'

import { emit } from '@nextcloud/event-bus'
import { generateRemoteUrl, generateUrl } from '@nextcloud/router'
import { getCurrentUser } from '@nextcloud/auth'
import { loadState } from '@nextcloud/initial-state'
import { showError, showSuccess } from '@nextcloud/dialogs'
import { translate } from '@nextcloud/l10n'
import axios from '@nextcloud/axios'

const userConfig = loadState('files', 'config', {
	show_hidden: false,
	crop_image_previews: true,
})

export default {
	name: 'Settings',
	components: {
		Clipboard,
		NcAppSettingsDialog,
		NcAppSettingsSection,
		NcCheckboxRadioSwitch,
		NcInputField,
		Setting,
	},

	props: {
		open: {
			type: Boolean,
			default: false,
		},
	},

	data() {
		return {

			...userConfig,

			// Settings API
			settings: window.OCA?.Files?.Settings?.settings || [],

			// Webdav infos
			webdavUrl: generateRemoteUrl('dav/files/' + encodeURIComponent(getCurrentUser()?.uid)),
			webdavDocs: 'https://docs.nextcloud.com/server/stable/go.php?to=user-webdav',
			webdavUrlCopied: false,
		}
	},

	beforeMount() {
		// Update the settings API entries state
		this.settings.forEach(setting => setting.open())
	},

	beforeDestroy() {
		// Update the settings API entries state
		this.settings.forEach(setting => setting.close())
	},

	methods: {
		onClose() {
			this.$emit('close')
		},

		setConfig(key, value) {
			emit('files:config:updated', { key, value })
			axios.post(generateUrl('/apps/files/api/v1/config/' + key), {
				value,
			})
		},

		async copyCloudId() {
			document.querySelector('input#webdav-url-input').select()

			if (!navigator.clipboard) {
				// Clipboard API not available
				showError(t('files', 'Clipboard is not available'))
				return
			}

			await navigator.clipboard.writeText(this.webdavUrl)
			this.webdavUrlCopied = true
			showSuccess(t('files', 'Webdav URL copied to clipboard'))
			setTimeout(() => {
				this.webdavUrlCopied = false
			}, 5000)
		},

		t: translate,
	},
}
</script>

<style lang="scss" scoped>

</style>
