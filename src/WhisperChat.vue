<template>
	<div>
		<h1>Whisper Chat App</h1>
		<div v-if="!configured">
			<symmetric-key-config @update-topic="updateTopic" @update-key="updateSymKey" :sym-key-id="symKeyId"></symmetric-key-config>

			username: <input v-model="name" /><br>
			<button @click="configWithKey" v-if="(symKeyId) && name">Start</button>
		</div> 
		<div v-else>

			<div >
				Key: {{symKeyId}}
			</div>
			<p v-for="m of msgs">
				<b>{{m.name}}</b>: {{m.text}}
			</p>
			<form @submit="sendMessage"
			>
			<input v-model="text" @keyup.enter="sendMessage" placeholder="Please type a message: "/>
			<button>Send</button>
			</form>
		</div>
	</div>
</template>

<script>
import Web3 from "web3";
import SymmetricKeyConfig from "./SymmetricKeyConfig.vue";
import { decodeFromHex, encodeToHex } from "./hexutils";

const defaultRecipientPubKey =
  "0x04ffb2647c10767095de83d45c7c0f780e483fb2221a1431cb97a5c61becd3c22938abfe83dd6706fc1154485b80bc8fcd94aea61bf19dd3206f37d55191b9a9c4";

export default {
  data() {
    this.web3 = new Web3(
      new Web3.providers.HttpProvider("http://localhost:8545")
    );
    this.shh = this.web3.shh;

    let data = {
      msgs: [],
      text: "",
      symKeyId: null,
      name: "",
      sympw: "",
      asym: true,
      configured: false,
      topic: null,
      recipientPubKey: defaultRecipientPubKey
    };

    return data;
  },

  components: { SymmetricKeyConfig },

  methods: {
    sendMessage() {
      let msg = {
        text: this.text,
        name: this.name
      };

      this.msgs.push(msg);

      let postData = {
        ttl: 7,
        powTarget: 2.01,
        powTime: 100,
        payload: encodeToHex(JSON.stringify(msg))
      };

      postData.symKeyID = this.symKeyId;
      postData.topic = this.topic;

      this.shh.post(postData);

      this.text = "";
    },

    updateSymKey(sympw) {
      this.shh
        .generateSymKeyFromPassword(sympw)
        .then(symKeyID => (this.symKeyId = symKeyID));
    },
    updateTopic(topic) {
      this.shh
        .generateSymKeyFromPassword(topic)
        .then(topic => (this.topic = topic));
    },

    configWithKey() {
      // TODO use a form
      if (!this.name || this.name.length == 0) {
        alert("Please pick a username");
        return;
      }

      let filter = {
        topics: [].push(this.topic)
      };

      if (!this.symKeyId || this.symKeyId.length == 0) {
        alert("please enter a pasword to generate a key!");
        return;
      }

      filter.symKeyID = this.symKeyId;

      this.msgFilter = this.shh.newMessageFilter(filter).then(filterId => {
        setInterval(() => {
          this.shh.getFilterMessages(filterId).then(messages => {
            for (let msg of messages) {
              let message = decodeFromHex(msg.payload);
              this.msgs.push({
                name: message.name,
                text: message.text
              });
            }
          });
        }, 1000);
      });

      this.configured = true;
    }
  }
};
</script>
