<template>
	<div>
		<h1>Whisper Example Chat Application</h1>
		<div v-if="!key">
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

function decodeFromHex(hex) {
	if (!hex || hex.length < 4 || hex[0] != "0" || hex[1] != "x" || hex.length % 2 != 0) {
		console.log(`Invalid hex string: ${hex}`);
		return "";
	} else {
		let result = "";

		for (let i = 2; i<hex.length; i+=2) {
			let n = parseInt(hex.slice(i, i+2), 16);
			result += String.fromCharCode(n);
		}

		try {
			return JSON.parse(result);
		} catch (e) {
			return "Error: message could not be decrypted";
		}
	}
}

function encodeToHex(string) {
	let hexEncodedMessage = "0x";

	try {
		for (let c of string) {
			hexEncodedMessage += c.charCodeAt(0).toString(16);
		}
	} catch(e) {
		
	}

	return hexEncodedMessage;
}

export default {
	data() {
		this.web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
		this.shh = this.web3.shh;

		let data = {
			msgs: [],
			text: "",
			keyEdit: null,
			key: null,
			sympw: "",
			name: ""
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

			this.shh.post({
				ttl: 7,
				topic: '0x07678231',
				powTarget: 2.01,
				powTime: 100,
				payload: encodeToHex(JSON.stringify(msg)),
				symKeyID: this.key
			});

			this.text = "";
		},

		updateKey() {
			this.shh.generateSymKeyFromPassword(this.sympw).then(symKeyID => this.keyEdit = symKeyID)
		},

		configWithKey() {
			// TODO use a form
			if (!this.sympw || this.sympw.length == 0) {
				alert("please enter a pasword to generate a key!");
				return;
			}

			if (!this.name || this.name.length == 0) {
				alert("Please pick a username");
				return;
			}

			this.key = this.keyEdit;

			this.msgFilter = this.shh.newMessageFilter({
				symKeyID: this.key,
				topics: ['0xdeadbeef']
			}).then(filterId => {
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
		}
	}
};
</script>
