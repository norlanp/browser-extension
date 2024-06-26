<template>
  <div class="content">
    <div class="p-3 container">
      <p>To get started, please follow these 3 simple steps:</p>

      <div class="mb-2">
        <span class="badge badge-primary badge-pill">1</span>
        Create your SimpleLogin account
        <a :href="apiUrl + '/auth/register'" target="_blank">here</a>
        if this is not already done.
      </div>

      <div class="mb-2">
        <span class="badge badge-primary badge-pill">2</span>
        Create and copy your
        <em>API Key</em>
        <a :href="apiUrl + '/dashboard/api_key'" target="_blank">here</a>.
      </div>

      <div class="mb-2">
        <span class="badge badge-primary badge-pill">3</span>
        Paste the
        <em>API Key</em> here 👇🏽
      </div>

      <input
        v-model="apiKey"
        v-on:keyup.enter="saveApiKey"
        placeholder="API Key"
        autofocus
        class="form-control mt-3 w-100"
      />

      <button @click="saveApiKey" class="btn btn-primary btn-block mt-2">
        Set API Key
      </button>
    </div>
  </div>
</template>

<script>
import SLStorage from "../SLStorage";
import EventManager from "../EventManager";
import Navigation from "../Navigation";
import Utils from "../Utils";
import { API_ROUTE } from "../APIService";

export default {
  data() {
    return {
      apiKey: "",
      apiUrl: "",
    };
  },
  async mounted() {
    this.apiUrl = await SLStorage.get(SLStorage.SETTINGS.API_URL);
  },
  methods: {
    async saveApiKey() {
      if (this.apiKey === "") {
        Utils.showError("API Key cannot be empty");
        return;
      }

      // check api key
      const res = await fetch(this.apiUrl + API_ROUTE.GET_USER_INFO.path, {
        headers: { Authentication: this.apiKey },
      });
      if (res.ok) {
        const apiRes = await res.json();
        const userName = apiRes.name || apiRes.email;
        await SLStorage.set(SLStorage.SETTINGS.API_KEY, this.apiKey);
        EventManager.broadcast(EventManager.EVENT.SETTINGS_CHANGED);

        Utils.showSuccess(`Hi ${userName}!`);
        Navigation.clearHistoryAndNavigateTo(Navigation.PATH.MAIN);
      } else {
        Utils.showError("Incorrect API Key.");
      }
    },
  },
  computed: {},
};
</script>
