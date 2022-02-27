<template>
	<div class="home-view">
		<header class="home__header">
			<div class="header__title">
				<h1 class="title__content">
					<span class="title__letter">B</span>ojkotuj
					<span class="title__letter">R</span>osyjskie
					<span class="title__letter">P</span>rodukty
				</h1>
				<img class="title__logo" src="~/assets/images/brp-logo.svg" />
			</div>
			<button class="header__button">Zaproponuj produkt</button>
		</header>
		<form class="home__add" @submit.prevent="addProduct">
			<label class="add__label">
				<input v-model="product.name" class="label__input" type="text" placeholder=" " required />
				<span class="label__placeholder">Nazwa</span>
			</label>
			<label class="add__label">
				<input v-model="product.cathegory" class="label__input" type="text" placeholder=" " required />
				<span class="label__placeholder">Kategoria</span>
			</label>
			<label class="add__label">
				<input v-model="product.subcathegory" class="label__input" type="text" placeholder=" " required />
				<span class="label__placeholder">Podkategoria</span>
			</label>
			<button type="submit">+</button>
		</form>
		<VueGoodTable
			v-if="admin"
			:columns="columns"
			:rows="rows"
			:search-options="{
				enabled: true
			}"
			@on-row-click="editRow($event.row)"
		/>
		<VueGoodTable
			v-else
			:columns="columns"
			:rows="rows"
			:search-options="{
				enabled: true
			}"
		/>
	</div>
</template>

<script lang="ts">
	import Vue from 'vue';
	import 'vue-good-table/dist/vue-good-table.css';

	interface Product {
		name: string;
		cathegory: string;
		subcathegory: string;
		valid: boolean;
		createdAt: number;
		id: string;
	}

	export default Vue.extend({
		name: 'IndexPage',
		components: {
			VueGoodTable: require('vue-good-table').VueGoodTable,
		},
		async asyncData({ $fire }) {
			const snap = await $fire.database.ref().child('products').get();
			return {
				// rows: snap.val().map((row: Product, key: string) => ({ ...row, id: key })).filter((row: Product) => row.valid),
				rows: Object.keys(snap.val()).map((key: string) => ({ ...snap.val()[key], id: key })),
			};
		},
		data() {
			return {
				admin: true,
				product: {
					name: '',
					cathegory: '',
					subcathegory: '',
				},
				columns: [
					{
						label: 'Nazwa',
						field: 'name',
					},
					{
						label: 'Kategoria',
						field: 'cathegory',
					},
					{
						label: 'Podkategoria',
						field: 'subcathegory',
					},
					{
						label: 'Data dodania',
						field: (row: Product) => this.$dayjs(row.createdAt).format('D MMMM YYYY (H:mm)'),
					},
				],
				rows: [],
			};
		},
		methods: {
			async addProduct(e: Event) {
				try {
					await this.$fire.database.ref('products').push().set({
						name: this.product.name,
						cathegory: this.product.cathegory,
						subcathegory: this.product.subcathegory,
						valid: false,
						createdAt: new Date().getTime(),
					});
					(e.target as HTMLFormElement).reset();
					alert('Produkt przesłano do weryfikacji');
				}
				catch (err) {
					console.error(err);
				}
			},
			editRow(row: Product) {
				console.log(row);
			},
		},
	});
</script>

<style lang="scss" scoped>
	.home-view {
		padding: 20px;
		.home__header {
			display: flex;
			align-items: center;
			justify-content: space-between;
			.header__title {
				display: flex;
				align-items: center;
				gap: 20px;
				.title__content {
					font-size: 1.5rem;
					.title__letter {
						font-size: 2rem;
						&:nth-child(1) {
							color: $flag-white;
						}
						&:nth-child(2) {
							color: $flag-blue;
						}
						&:nth-child(3) {
							color: $flag-red;
						}
					}
				}
				.title__logo {
					width: 50px;
					height: 50px;
				}
			}
			.header__button {
				background-color: $bg-secondary;
				border: none;
				border-radius: 10px;
				padding: 10px 15px;
				font-weight: bold;
				transition: 0.2s ease;
				@include hover {
					background-color: $bg-tertiary;
				}
			}
		}
		.home__add {
			display: flex;
			gap: 10px;
			margin: 20px 0;
			.add__label {
				position: relative;
				.label__select {
					pointer-events: none;
				}
				.label__select,
				.label__input {
					width: 100%;
					padding: 15px 30px;
					background-color: $bg-secondary;
					border: 0;
					border-radius: 10px;
					outline: none;
					transition: 0.2s ease;
					&:focus,
					&:not(:placeholder-shown) {
						~ .label__placeholder {
							top: 0px;
							left: 10px;
							font-size: 0.75rem;
						}
					}
					&:focus {
						~ .label__placeholder {
							background-color: $bg-tertiary;
						}
						background-color: $bg-tertiary;
					}
				}
				.label__placeholder {
					position: absolute;
					top: 50%;
					left: 25px;
					padding: 5px;
					transform: translateY(-50%);
					background-color: $bg-secondary;
					border-radius: 5px;
					pointer-events: none;
					transition: 0.2s ease;
					color: $text-primary;
				}
				.label__list {
					display: none;
					position: absolute;
					top: calc(100% + 5px);
					left: 0;
					width: 100%;
					background-color: $bg-primary;
					border-radius: 10px;
					z-index: 3;
					overflow: hidden;
					.list__item {
						padding: 15px 30px;
						transition: 0.2s ease;
						// @include hover {
						// 	background-color: darken($bg-primary, 10);
						// }
					}
				}
			}
		}
	}
</style>
<style lang="scss">
	.vgt-wrap {
		border-radius: 10px;
		overflow: hidden;
		.vgt-responsive {
			height: calc(100vh - 250px);
			background-color: $bg-secondary;
		}
	}
</style>
