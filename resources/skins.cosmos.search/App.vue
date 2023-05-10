<template>
	<cdx-typeahead-search
		:id="id"
		ref="searchForm"
		:class="rootClasses"
		:search-results-label="$i18n( 'searchresults' ).text()"
		:accesskey="searchAccessKey"
		:title="searchTitle"
		:placeholder="searchPlaceholder"
		:aria-label="searchPlaceholder"
		:initial-input-value="searchQuery"
		:button-label="$i18n( 'searchbutton' ).text()"
		:form-action="action"
		:show-thumbnail="showThumbnail"
		:highlight-query="highlightQuery"
		:auto-expand-width="autoExpandWidth"
		:search-results="suggestions"
		:search-footer-url="searchFooterUrl"
		@input="onInput"
	>
		<template #default>
			<input
				type="hidden"
				name="title"
				:value="searchPageTitle"
			>
			<input
				type="hidden"
				name="wprov"
				:value="wprov"
			>
		</template>
		<!-- eslint-disable-next-line vue/no-template-shadow -->
		<template #search-footer-text="{ searchQuery }">
			<span v-i18n-html:cosmos-searchsuggest-containing="[ searchQuery ]"></span>
		</template>
	</cdx-typeahead-search>
</template>

<script>
/* global SearchSubmitEvent */
const { CdxTypeaheadSearch } = require( '@wikimedia/codex-search' ),
	{ defineComponent, nextTick } = require( 'vue' ),
	restClient = require( './restSearchClient.js' ),
	actionClient = require( './actionSearchClient.js' );

// @vue/component
module.exports = exports = defineComponent( {
	name: 'App',
	components: { CdxTypeaheadSearch },
	props: {
		id: {
			type: String,
			required: true
		},
		searchPageTitle: {
			type: String,
			default: 'Special:Search'
		},
		autofocusInput: {
			type: Boolean,
			default: false
		},
		action: {
			type: String,
			default: ''
		},
		/** The keyboard shortcut to focus search. */
		// eslint-disable-next-line vue/require-default-prop
		searchAccessKey: {
			type: String
		},
		/** The access key informational tip for search. */
		// eslint-disable-next-line vue/require-default-prop
		searchTitle: {
			type: String
		},
		/** The ghost text shown when no search query is entered. */
		// eslint-disable-next-line vue/require-default-prop
		searchPlaceholder: {
			type: String
		},
		/**
		 * The search query string taken from the server-side rendered input immediately before
		 * client render.
		 */
		// eslint-disable-next-line vue/require-default-prop
		searchQuery: {
			type: String
		},
		showThumbnail: {
			type: Boolean,
			// eslint-disable-next-line vue/no-boolean-default
			default: true
		},
		showDescription: {
			type: Boolean,
			// eslint-disable-next-line vue/no-boolean-default
			default: true
		},
		highlightQuery: {
			type: Boolean,
			// eslint-disable-next-line vue/no-boolean-default
			default: true
		},
		autoExpandWidth: {
			type: Boolean,
			default: false
		}
	},
	data() {
		return {
			// Suggestions to be shown in the TypeaheadSearch menu.
			suggestions: [],

			// Link to the search page for the current search query.
			searchFooterUrl: '',

			// Whether to apply a CSS class that disables the CSS transitions on the text input
			disableTransitions: this.autofocusInput
		};
	},
	computed: {
		rootClasses() {
			return {
				'cosmos-search-box-disable-transitions': this.disableTransitions
			};
		}
	},
	methods: {
		/**
		 * Fetch suggestions when new input is received.
		 *
		 * @param {string} value
		 */
		onInput: function ( value ) {
			const domain = mw.config.get( 'wgCosmosSearchHost', location.host ),
				query = value.trim();

			this.currentSearchQuery = query;

			if ( query === '' ) {
				this.suggestions = [];
				this.searchFooterUrl = '';
				return;
			}

			if ( mw.config.get( 'wgCosmosSearchUseActionAPI', false ) ) {
				actionClient( mw.config ).fetchByTitle( query, domain, 10 ).fetch
					.then( ( data ) => {
						this.suggestions = data.results;
						this.searchFooterUrl = urlGenerator.generateUrl( query );
						const event = {
							numberOfResults: data.results.length,
							query: query
						};
					} )
					.catch( () => {
						// TODO: error handling
					} );
					return;
			}

			restClient( mw.config ).fetchByTitle( query, domain, 10 ).fetch
				.then( ( data ) => {
					this.suggestions = data.results;
					this.searchFooterUrl = urlGenerator.generateUrl( query );
					const event = {
						numberOfResults: data.results.length,
						query: query
					};
				} )
				.catch( () => {
					// TODO: error handling
				} );
		}
	},
	mounted() {
		if ( this.autofocusInput ) {
			this.$refs.searchForm.focus();
			nextTick( () => {
				this.disableTransitions = false;
			} );
		}
	}
} );
</script>
