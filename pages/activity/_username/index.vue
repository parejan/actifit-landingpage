<template>
  <!-- activity report listing for single user -->
  <div>
    <NavbarBrand />

    <!-- listing -->
    <div class="container pt-5 mt-5 pb-5">
      <h2 class="text-center mb-5">{{ $t('Activity_Reports_by') }} {{ username }}</h2>
	  <!--<ChainSelection />-->
	  
      <!-- show spinner while loading -->
      <div class="text-center" v-if="loading">
        <i class="fas fa-spinner fa-spin text-brand"></i>
      </div>

      <!-- show listing when loaded -->
	  <div class="row" v-if="userReports.length">
		<div class="row"  v-for="iterx in Math.ceil(userReports.length / splitFactor)" :key="iterx">
			<div v-for="itery in splitFactor" :key="itery" class="col-md-6 col-lg-4 mb-4">
				<Report v-if="(iterx - 1) * splitFactor + (itery - 1) < userReports.length" :report="userReports[(iterx - 1) * splitFactor + (itery - 1)]" />
			</div>
			<div class="col-md-6 col-lg-4 mb-4" v-if="(iterx - 1) < inlineAds">
				<adsbygoogle ad-slot="7038919015" ad-format="fluid" ad-layout-key="-fb+5w+4e-db+86"/>
			</div>
		</div>
      </div>

      <!-- or no content message when no reports found -->
      <div class="text-center text-muted" v-if="!userReports.length && !loading">
        {{ username }} {{ $t('error_no_tracked_activity') }}
      </div>

      <!-- show load more button if there are more posts available -->
      <div class="text-center" v-if="moreUserReportsAvailable">
        <a href="#" class="btn btn-brand" @click.prevent="loadMore()">
          {{ $t('load_more') }}
          <i class="fas fa-spinner fa-spin" v-if="loadingMore"></i>
        </a>
      </div>
    </div>

    <Footer />
    <ReportModal :report="activeReport" />
	<DailyActivityChartModal :report="activeReport" />
    <EditReportModal />
    <VoteModal />
    <no-ssr>
      <div>
        <notifications :group="'success'" :position="'top center'" :classes="'vue-notification success'" />
        <notifications :group="'error'" :position="'top center'" :classes="'vue-notification error'" />
      </div>
    </no-ssr>
  </div>
</template>

<script>
  import NavbarBrand from '~/components/NavbarBrand'
  import Report from '~/components/Report'
  import Footer from '~/components/Footer'
  import ReportModal from '~/components/ReportModal'
  import EditReportModal from '~/components/EditReportModal'
  import VoteModal from '~/components/VoteModal'
  import DailyActivityChartModal from '~/components/DailyActivityChartModal'

  import { mapGetters } from 'vuex'
  //import ChainSelection from '~/components/ChainSelection'

  export default {
    head () {
		return {
		  title: `Actifit activity reports by ${this.username} - Actifit.io`,
		  meta: [
			{ hid: 'description', name: 'description', content: `Listing of activity reports by ${this.username}` },
			{ hid: 'ogdescription', name: 'og:description', content: `Listing of activity reports by ${this.username}` }
		  ]
		}
	},
	components: {
      NavbarBrand,
      Report,
      Footer,
      ReportModal,
      EditReportModal,
      VoteModal,
	  DailyActivityChartModal //,
	  //ChainSelection
    },
    data () {
      return {
        loading: true, // initial loading state
        loadingMore: false, // loading state for loading more reports
		splitFactor: 9,
		inlineAds: 2,
      }
    },
	watch: {
	  user: 'fetchUserData',
	  bchain: async function(newBchain) {
		console.log('user activity change in chain '+newBchain);
		this.cur_bchain = newBchain;
		await this.$store.dispatch('steemconnect/refreshUser');
		
		// reset previously fetched posts to get latest
		this.$store.commit('setUserReports', [])

		// disable load more button and only show if there actually are more posts to load
		this.$store.commit('setMoreUserReportsAvailable', false)
		this.$store.dispatch('fetchUserReports', this.username)
		//this.reload += 1;
	  }
	},
    computed: {
	  ...mapGetters('steemconnect', ['user']),
	  ...mapGetters('steemconnect', ['stdLogin']),
      ...mapGetters(['userReports', 'moreUserReportsAvailable', 'activeReport', 'bchain']),

      // get username from url
      username () {
	    if (this.$route.params.username.startsWith('@')){
		  return this.$route.params.username.substring(1, this.$route.params.username.length);
		}
        return this.$route.params.username
      }
    },
    methods: {
      async loadMore () {
        this.loadingMore = true
		
		let cur_bchain = (localStorage.getItem('cur_bchain')?localStorage.getItem('cur_bchain'):'HIVE');
		this.$store.commit('setBchain', cur_bchain);
	  
        await this.$store.dispatch('fetchUserReports', this.username)
        this.loadingMore = false
      },
	  async fetchUserData () {
		if (typeof this.user != 'undefined' && this.user != null){	  
		  
		  if (!localStorage.getItem('std_login')){
		  //if (!this.stdLogin){
			  //update user info from blockchain
			  let user_data = await this.$steemconnect.me();
			  this.user.account = user_data.account;
		  }
		  //ensure we fetch proper logged in user data
		  this.$store.dispatch('fetchUserTokens')
		  this.$store.dispatch('fetchUserRank')
		}
	  },
    },
    async mounted () {
      // login
      this.$store.dispatch('steemconnect/login')
	  this.fetchUserData();
	  
      // reset previously fetched posts to get latest
      this.$store.commit('setUserReports', [])

      // disable load more button and only show if there actually are more posts to load
      this.$store.commit('setMoreUserReportsAvailable', false)

      // fetch reports
	  
	  let cur_bchain = (localStorage.getItem('cur_bchain')?localStorage.getItem('cur_bchain'):'HIVE');
	  this.$store.commit('setBchain', cur_bchain);
	  
      await this.$store.dispatch('fetchUserReports', this.username)

      // remove loading indicator
      this.loading = false
    }
  }
</script>
