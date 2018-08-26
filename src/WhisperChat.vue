<template>
	<div>
		<h1>Whisper Example Chat Application</h1>
		<div v-if="!configured">
			<symmetric-key-config  @update-key="updateSymKey" :sym-key-id="symKeyId"></symmetric-key-config>

			username: <input v-model="name" /><br>
			<button @click="configWithKey" v-if="name">Start</button>
		</div>
		<div v-else>
			<div >
				Key: {{symKeyId}}
			</div>
			<p v-for="m of msgs">
				<b>{{m.name}}</b>: {{m.text}}
			</p>
			Please type a message: <input v-model="text" @keyup.enter="sendMessage" />
			<button @click="sendMessage">Send</button>
		</div>
	</div>
</template>

<script>
import Web3 from "web3";
import SymmetricKeyConfig from "./SymmetricKeyConfig.vue";
import { decodeFromHex, encodeToHex } from "./hexutils";

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
      asymKeyId: null,
      sympw: "",
      asym: true,
      configured: false,
      topic: "0x5a4ea131",
      asymPubKey: ""
    };

    this.shh
      .newKeyPair()
      .then(id => {
        data.asymKeyId = id;
        return this.shh
          .getPublicKey(id)
          .then(pubKey => (this.asymPubKey = pubKey))
          .catch(console.log);
      })
      .catch(console.log);

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
        topic: "0x07678231",
        powTarget: 2.01,
        powTime: 100,
        payload: encodeToHex(JSON.stringify(msg))
      };

      postData.symKeyID = this.symKeyId;

      this.shh.post(postData);

      this.text = "";
    },

    updateSymKey(sympw) {
      this.shh
        .generateSymKeyFromPassword(sympw)
        .then(symKeyID => (this.symKeyId = symKeyID));
    },
    updateTopic(topic) {
      this.topic = "0x07678231";
    },

    configWithKey() {
      // TODO use a form
      if (!this.name || this.name.length == 0) {
        alert("Please pick a username");
        return;
      }

      if (!this.topic || this.topic.length == 0) {
        alert("Please enter a topic");
        return;
      }

      let filter = {
        topics: ["0x07678231"]
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
