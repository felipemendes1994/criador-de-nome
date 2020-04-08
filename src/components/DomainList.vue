<template>
	<div>
		<div id="main">
			<div class="container">
				<div class="row">
					<div class="col-md">
						<AppItemList title="Prefixos" type="prefix" v-bind:items="items.prefix" v-on:addItem="addItem" v-on:deleteItem="deleteItem"></AppItemList>
					</div>
					<div class="col-md">
						<AppItemList title="Sufixos" type="sufix" v-bind:items="items.sufix" v-on:addItem="addItem" v-on:deleteItem="deleteItem"></AppItemList>
					</div>
				</div>
				<br />
				<h5>Domains <span class="badge badge-info">{{ domains.length }}</span></h5>
				<div class="card">
					<div class="card-body">
						<ul class="list-group">
							<li class="list-group-item" v-for="domain in domains" v-bind:key="domain.name">
								<div class="row">
									<div class="col-md-6">{{ domain.name }}</div>
									<div class="col-md-3"><span v-bind:class="domain.classes">{{ domain.available }}</span></div>
									<div class="col-md-3 text-right">
										<a class="btn btn-info" v-bind:href="domain.checkout" target="_blank"><span
												class="fa fa-shopping-cart"></span></a>
									</div>
								</div>
							</li>
						</ul>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
import "bootstrap/dist/css/bootstrap.css";
import "font-awesome/css/font-awesome.css";
import AppItemList from "./AppItemList";
import axios from "axios/dist/axios";

export default {
	name: "App",
	components: {
		AppItemList
	},
	data: function() {
		return {
			items: {
				prefix: [],
				sufix: []
			},
			domains: []
		};
	},
	methods: {
		addItem(item){
			axios({
				url: "http://localhost:4000",
				method: "post",
				data: {
					query: `
						mutation ($item: ItemInput) {
							item: saveItem(item: $item){
								id,
								type,
								description
							}
						}
					`,
					variables: {
						item
					}
				}
			}).then(response => {
				const query = response.data;
				const item = query.data.item;
				this.items[item.type].push(item);
				this.generateDomains();
			});
		},
		deleteItem(item){
			axios({
				url: "http://localhost:4000",
				method: "post",
				data: {
					query: `
						mutation ($id: Int) {
							deleteItem(id: $id)
						}
					`,
					variables: {
						id: item.id
					}
				}
			}).then(() => {
				this.items[item.type].splice(this.items[item.type].indexOf(item), 1);
				this.generateDomains();
			});
		},
		getItems(type){
			return axios({
				url:"http://localhost:4000",
				method: "post",
				data: {
					query: `
						query ($type: String) {
							items: items (type: $type) {
								id
								type
								description
							}
						}
					`,
					variables:{
						type
					}

				}
			}).then(response => {
				this.items[type] = response.data.data.items;
			});
		},
		generateDomains(){
			return axios({
				url:"http://localhost:4000",
				method: "post",
				data: {
					query: `
						mutation {
							domains: generateDomains {
								name
								checkout
								available
							}
						}
					`

				}
			}).then(response => {
				const query = response.data;
				this.domains = query.data.domains.map(domain => {
					return {
						name: domain.name,
						checkout: domain.checkout,
						available: (domain.available) ? "Disponível" : "Indisponível",
						classes: (domain.available) ? "badge badge-success" : "badge badge-secondary"
					};
				});
			});
		}
	},
	created(){
		Promise.all([
			this.getItems("prefix"),
			this.getItems("sufix")
		]).then(() => {
			this.generateDomains();
		});
	}
};

</script>

<style>
#slogan {
  margin-top: 30px;
  margin-bottom: 30px;
}

#main {
  background-color: #cccccc;
  padding-top: 30px;
  padding-bottom: 30px;
}
</style>