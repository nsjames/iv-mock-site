<template>
	<div class="home">
		<pre v-if="wallet && wallet.account">{{wallet.account.id}} -- {{wallet.account.publicKey}}</pre>
		<section class="actions">
			<button @click="login">Login</button>
			<button @click="logout">Logout</button>
			<button @click="addAccount">Add account</button>
			<button @click="transfer">Transfer</button>
			<button @click="multiTransfer">Multi Transfer</button>
			<button @click="customTokenTransfer">Custom token Transfer</button>
			<button @click="createToken">Create Token</button>
		</section>
		<hr />

		<pre>{{logs}}</pre>
	</div>


</template>

<script>
	const {
		Client, PrivateKey, AccountCreateTransaction,
		AccountBalanceQuery, Hbar, TransferTransaction, Transaction,
		TokenCreateTransaction,
		TokenAssociateTransaction
	} = require("@hashgraph/sdk");

	export default {
		data(){return {
			wallet:null,
			logs:'',
		}},
		methods:{
			addLog(msg){
				if(typeof msg === 'object') msg = JSON.stringify(msg, null, 4);
				this.logs += `\r\n${msg}`;
			},
			async login(){
				this.addLog(await this.wallet.login('Testnet').catch(e => e))
			},
			async logout(){
				this.addLog(await this.wallet.logout().catch(e => e));
			},
			async addAccount(){
				const key = await PrivateKey.generate();
				const network = 'Testnet';
				const id = '0.0.304197';
				this.addLog(await this.wallet.addAccount(network, id, key.toString()).catch(e => e));
			},
			async transfer(){
				const client = Client.forTestnet();
				client.setOperatorWith(
					this.wallet.account.id,
					this.wallet.account.publicKey,
					this.provider
				)
				const response = await new TransferTransaction()
					.addHbarTransfer(this.wallet.account.id, Hbar.fromTinybars(-1))
					.addHbarTransfer('0.0.304192', Hbar.fromTinybars(1))
					.signWithOperator(client).catch(err => {
						this.addLog(err);
						return null;
					})
				// .execute(client);
				if(response){
					console.log('transferTransactionResponse', response);
					this.addLog(`Check console for transaction log, too long.`)
					const executed = await response.execute(client);
					console.log(executed);
					console.log(await executed.getReceipt(client));
				}
			},
			async multiTransfer(){
				const client = Client.forTestnet();
				client.setOperatorWith(
					this.wallet.account.id,
					this.wallet.account.publicKey,
					this.provider
				)
				const response = await new TransferTransaction()
					.addHbarTransfer(this.wallet.account.id, Hbar.fromTinybars(-3))
					.addHbarTransfer('0.0.304192', Hbar.fromTinybars(1))
					.addHbarTransfer('0.0.304193', Hbar.fromTinybars(1))
					.addHbarTransfer('0.0.304194', Hbar.fromTinybars(1))
					.signWithOperator(client).catch(err => {
						this.addLog(err);
						return null;
					})
				// .execute(client);
				if(response){
					console.log('transferTransactionResponse', response);
					this.addLog(`Check console for transaction log, too long.`)
					const executed = await response.execute(client);
					console.log(executed);
					console.log(await executed.getReceipt(client));
				}
			},
			async customTokenTransfer(){
				const client = Client.forTestnet();
				client.setOperatorWith(
					this.wallet.account.id,
					this.wallet.account.publicKey,
					this.provider
				)

				// const associate = await new TokenAssociateTransaction()
				// 	.setTokenIds(['0.0.307194'])
				// 	.setAccountId(this.wallet.account.id)
				// 	.execute(client);

				const response = await new TransferTransaction()
					.addTokenTransfer('0.0.307194', this.wallet.account.id, -1)
					.addTokenTransfer('0.0.307194', '0.0.304192' /* some random dude who has a bunch of my HBAR */, 1)
					.signWithOperator(client).catch(err => {
						this.addLog(err);
						return null;
					})
				// .execute(client);
				if(response){
					console.log('transferTransactionResponse', response);
					this.addLog(`Check console for transaction log, too long.`)
					const executed = await response.execute(client);
					console.log(executed);
					console.log(await executed.getReceipt(client));
				}
			},
			async createToken(){
				const client = Client.forTestnet();
				const adminKey = await PrivateKey.generate();
				client.setOperatorWith(
					this.wallet.account.id,
					this.wallet.account.publicKey,
					this.wallet.getTransactionSigner()
				)
				const response = await new TokenCreateTransaction()
					.setTokenName("Your Token Name")
					.setTokenSymbol("F")
					.setTreasuryAccountId(this.wallet.account.id)
					.setInitialSupply(5000)
					.setAdminKey(adminKey)
					.freezeWith(client)
					.signWithOperator(client).catch(err => {
						this.addLog(err);
						return null;
					})
				// .execute(client);
				if(response){
					response.sign(adminKey);
					console.log('response', response);
					this.addLog(`Check console for transaction log, too long.`)
					const executed = await response.execute(client);
					console.log(executed);
					console.log(await executed.getReceipt(client));
				}
			}
		},
		mounted(){
			document.addEventListener('hederaWalletLoaded', async () => {
				this.wallet = window.wallet;
				this.provider = this.wallet.getTransactionSigner();
			})
		}
	}
</script>
