<template>
	<div>
		<h1>Whisper Example Chat Application</h1>
		<div v-if="!configured">
			Key generation password: <input v-model="sympw" @input="updateKey" /><br>
			username: <input v-model="name" /><br>
			Key: {{keyEdit}}<br>
			<button @click="configWithKey">Start</button>
		</div>
		<div v-else>
			Key: {{key}}<br>
			<p v-for="m of msgs">
				<b>{{m.name}}</b>: {{m.text}}
			</p>
			Please type a message: <input v-model="text" @keyup.enter="sendMessage" />
			<button @click="sendMessage">Send</button>
		</div>
	</div>
</template>

<script>
import Web3 from 'web3';
import {decodeFromHex, encodeToHex} from './hexutils';

const defaultRecipientPubKey = "0x04ffb2647c10767095de83d45c7c0f780e483fb2221a1431cb97a5c61becd3c22938abfe83dd6706fc1154485b80bc8fcd94aea61bf19dd3206f37d55191b9a9c4";
const defaultTopic = "0x5a4ea131";

export default {
	data() {
		this.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
		this.shh = this.web3.shh;

		let data = {
			msgs: [],
			text: "",
			keyEdit: null,
			key: null,
			name: "",
			sympw: "",
			configured: false,
			topic: defaultTopic,
			recipientPubKey: defaultRecipientPubKey
		};

		return data;
	},

	methods: {
		sendMessage() {
			let msg = {
				text: this.text,
				name: this.name
			};

			this.msgs.push(msg);

			let postData = {
				ttl: 7,
				topic: '0x07678231',
				powTarget: 2.01,
				powTime: 100,
				payload: encodeToHex(JSON.stringify(msg)),
				postData.symKeyID = this.keyEdit;

			this.shh.post(postData);

			this.text = "";
		},

		updateKey() {
			this.shh.generateSymKeyFromPassword(this.sympw).then(symKeyID => this.keyEdit = symKeyID)
		},

		configWithKey() {
			// TODO use a form
			if (!this.name || this.name.length == 0) {
				alert("Please pick a username");
				return;
			}

			let filter = {
				topics: ['0xdeadbeef']
			};
				return;
			}

				if (!this.keyEdit || this.keyEdit.length == 0) {
					alert("please enter a pasword to generate a key!");
				return;
			}

			this.key = this.keyEdit;
				filter.symKeyID = this.keyEdit;

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
