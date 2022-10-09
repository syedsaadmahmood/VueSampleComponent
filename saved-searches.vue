<template>
	<div class="grid multi-col-layout" :class="gridSettings['cols']" ref="wrapper">
		<div class="column column-start" :class="gridSettings['start']">
			<div class="bg-gray-0 p-2 lg:p-3 rounded-md">
				<h2 class="section-headline">Tailor your search</h2>
				<select-list-box class="mt-2"
								:title="'Tags'"
								:items="searchTags"
								@change="filterTags"
								@deselectAll="emptyTags">
				</select-list-box>
			</div>
		</div>
		<div class="p-2 column column-end" :class="gridSettings['end']">
			<div class="flex justify-between mt-0 md:mt-2 lg:mt-0 section-headline mb-3">
				<p class="flex-1">Saved searches</p>

				<button class="button _outline blue small rounded ml-2" @click="createSearch">
					<span class="mdi mdi-plus"></span>
					Create Search
				</button>
			</div>

			<saved-search-list :saved-searches="savedSearches"
							   :pagination="searchesPagination"
							   @deleteSearch="deleteSavedSearch($event)"
							   @editSearch="editSavedSearch($event)"
							   @applySearch="applySavedSearch($event)">
			</saved-search-list>
		</div>
	</div>
</template>



<script>
	import selectTagListBox from 'Src/components/selectTagListBox';
	import savedSearchList from 'Src/components/parts/savedSearch/savedSearchList';
	import selectListBox from 'Src/components/selectListBox.vue';

	import ensureUrlParamsMixin from 'Src/mixins/ensureUrlParamsMixin';
	import { mergeFiltersWithQuery } from '~/utils';


	export default {
		name: 'SavedSearches',
		middleware: ['auth'],
		mixins: [ensureUrlParamsMixin],
		components: {
			selectTagListBox,
			savedSearchList,
			selectListBox,
		},

		data () {
			return {
				gridSettings: {
					cols: 'grid-cols-3',
					start: 'col-start-1 col-end-4 lg:col-end-2',
					end: 'col-start-1 col-end-4 lg:col-start-2',
				}
			}
		},

		asyncData ({ store, route, params, query }) {
			let filters = mergeFiltersWithQuery({ skipRefresh: true, evaluations: [4], affectedFlows: [1] }, route.query);
			store.commit('filters/setState', filters);
			store.commit('filters/excludeFilters', ['affectedJurisdictions', 'affectedFlows']);
			return store.dispatch('pages/fetchSavedSearchesAsyncData', { initial: true });
		},

		mounted () {
			new ResizeObserver(this.gridNumber).observe(this.$refs.wrapper);
			window.addEventListener('resize', this.gridNumber);
			this.gridNumber();
			this.syncFiltersToQueryParams();
		},

		methods: {
			editSavedSearch (searchId) {
				this.$nuxt.$emit('editSavedSearch', searchId);
			},

			deleteSavedSearch(searchId) {
				this.$nuxt.$emit('deleteSavedSearch', searchId);
			},

			applySavedSearch (searchId) {
				this.$nuxt.$emit('applySavedSearch', searchId);
			},

			createSearch () {
				this.$store.commit('sidebar/setSidebarState', { isOpened: true, isSaveSearchVisible: true });
			},

			filterTags(tags) {
				let filterTags = tags.map(i => i.id)
				let params = { filterTags }
				this.$store.commit('searches/setValue', { key: 'searchesOffset', value: 0 });
				this.$store.commit('searches/setFiltertags', filterTags);
				this.$store.dispatch('pages/fetchSavedSearchesAsyncData', { initial: false, params });
			},

			emptyTags() {
				this.$store.commit('searches/setFiltertags', []);
				this.$store.dispatch('pages/fetchSavedSearchesAsyncData', { initial: false });
			},
			gridNumber() {
				const threshold = 1000;
				let width = 0;
				if (this.$refs.wrapper) {
					width = this.$refs.wrapper.clientWidth;
				}
				if (width < threshold) {
					this.gridSettings['cols'] =  ' grid-cols-1';
					this.gridSettings['start'] =  ' col-start-1 col-end-2 even-padding';
					this.gridSettings['end'] =  ' col-start-1 col-end-2 even-padding';
				} else {
					this.gridSettings['cols'] =  ' grid-cols-3';
					this.gridSettings['start'] =  ' col-start-1 col-end-4 lg:col-end-2';
					this.gridSettings['end'] =  ' col-start-1 col-end-4 lg:col-start-2';
				}
			}
		},

		computed: {
			searchTags () {
				return this.$store.state.searches.tags;
			},

			savedSearches () {
				return this.$store.state.searches.savedSearches;
			},

			searchesPagination () {
				return {
					limit: this.$store.state.searches.searchesLimit,
					offset: this.$store.state.searches.searchesOffset,
					count: this.$store.state.searches.searchesCount,
				}
			},
		}
	};
</script>



<style lang="scss" scoped>
	.saved-searches {
		padding: 40px;

		.search-wrapper {
			display: flex;
			align-items: center;
		}

		.items {
			display: flex;
			align-items: flex-start;
			gap: 2rem;

			.tag-filter {
				flex: 0 0 300px;

				&.select-list-box {
					width: 95%;
					margin: 20px auto;
					padding: 5px 20px;
					box-shadow: none !important;
					border: 1px solid var(--v-shades3);
				}
			}
		}
	}
</style>
