<template>
<form novalidate class="md-layout">
  <md-card class="md-layout-item md-size-30 md-small-size-50">
    <md-card-header>
          <div class="md-title"><h3>SignalR Anlık Veri Akışı</h3></div>
        </md-card-header>
  <md-card-content>
    <form>
      <md-field>
      <md-input type="text" name="newValue" placeholder="Yeni Mesaj ..." id="newValue" v-model="newValue" />
      
      <md-button class="md-raised md-primary" @click="addValue">Gönder</md-button>
      </md-field>
    </form>
    <br>
    <p>Son Mesajlar</p>
    <ul v-for="v in values" :key="v">
      <li>
        <b>{{v}}</b> - <button @click="deleteValue(v)">Sil</button>
      </li>
    </ul>
  </md-card-content>
  </md-card>
  </form>
</template>

<script lang="ts">
import Vue from "vue";
import { HubConnection, TransportType } from "@aspnet/signalr";
const apiBase = "http://192.168.1.107:5000";
const errMessage = "Api bağlantısı sağlanamadı";
export default Vue.extend({
  data: () => ({
    values: [],
    newValue: ""
  }),

  
  beforeMount: async function() {
    try {
      const response = await fetch(`${apiBase}/Message`);
      const values: string[] = await response.json();
      this.values = values;
      const hubConnection = new HubConnection(`${apiBase}/Hubs/Message`, {
        transport: TransportType.WebSockets
      });
      hubConnection.on("Add", (value: string) => {
        if(value != null)
        this.values.push(value);
      });
      hubConnection.on("Delete", (value: string) => {
        this.values = this.values.filter(v => v !== value);
      });
      hubConnection.start();
    } catch {
      alert(errMessage);
    }
  },

  methods: {
    addValue: async function() {
      try {
        const response = await fetch(`${apiBase}/Message`, {
          body: JSON.stringify(this.newValue),
          headers: new Headers({
            "content-type": "application/json"
          }),
          method: "POST"
        });
        this.newValue = "";
      } catch {
        alert(errMessage);
      }
    },
    deleteValue: async function(value) {
      try {
        const response = await fetch(`${apiBase}/Message/${this.values.indexOf(value)}`, {
          method: "DELETE"
        });
      } catch {
        alert(errMessage);
      }
    }
  }
});
</script>

<style scoped>
  li {
    list-style: none;
  }

  .md-progress-bar {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
  }
</style>
