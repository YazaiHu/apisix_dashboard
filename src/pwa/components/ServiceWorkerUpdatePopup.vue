<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component({
  name: 'ServiceWorkerUpdatePopup'
})
export default class extends Vue {
  private refreshing = false
  private notificationText = 'New content is available!'
  private refreshButtonText = 'Refresh'
  private registration: ServiceWorkerRegistration | null = null

  created() {
    // Listen for swUpdated event and display refresh notification as required.
    document.addEventListener('swUpdated', this.showRefreshUI, { once: true })
    // Refresh all open app tabs when a new service worker is installed.
    navigator.serviceWorker.addEventListener('controllerchange', () => {
      if (this.refreshing) return
      this.refreshing = true
      window.location.reload()
    })
  }

  render() {
    // Avoid warning for missing template
  }

  private showRefreshUI(e: Event) {
    // Display a notification inviting the user to refresh/reload the app due
    // to an app update being available.
    // The new service worker is installed, but not yet active.
    // Store the ServiceWorkerRegistration instance for later use.
    const h = this.$createElement
    this.registration = (e as CustomEvent).detail
    this.$notify.info({
      title: 'Update available',
      message: h('div', { class: 'sw-update-popup' }, [
        this.notificationText,
        h('br'),
        h('button', {
          on: {
            click: (e: Event) => {
              e.preventDefault()
              this.refreshApp()
            }
          }
        }, this.refreshButtonText)
      ]),
      position: 'bottom-right',
      duration: 0
    })
  }

  private refreshApp() {
    // Protect against missing registration.waiting.
    if (!this.registration || !this.registration.waiting) return
    this.registration.waiting.postMessage('skipWaiting')
  }
}
</script>

<style lang="scss" scoped>
.sw-update-popup > button {
  margin-top: 0.5em;
  padding: 0.25em 1.5em;
}
</style>
