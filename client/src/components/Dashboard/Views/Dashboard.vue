<template>
  <div>
    <!--Stats cards-->
    <div class="row">
      <div class="col-lg-3 col-sm-6" v-for="stats in statsCards">
        <stats-card>
          <div class="icon-big text-center" :class="`icon-${stats.type}`" slot="header">
            <i :class="stats.icon"></i>
          </div>
          <div class="numbers" slot="content">
            <p>{{stats.title}}</p>
            {{stats.value}}
          </div>
          <div class="stats" slot="footer">
            <i :class="stats.footerIcon"></i> {{stats.footerText}}
          </div>
        </stats-card>
      </div>
    </div>
  </div>
</template>
<script>
  import StatsCard from 'components/UIComponents/Cards/StatsCard.vue'
  import ChartCard from 'components/UIComponents/Cards/ChartCard.vue'
  export default {
    components: {
      StatsCard,
      ChartCard
    },
    /**
     * Chart data used to render stats, charts. Should be replaced with server data
     */
    data () {
      return {
        statsCards: [
          {
            type: 'success',
            icon: 'ti-money',
            title: 'Money',
            value: '0',
            footerText: '',
            footerIcon: 'ti-wallet'
          },
          {
            type: 'info',
            icon: 'ti-server',
            title: 'Blocks height',
            value: '0',
            footerText: 'Difficulty: 1, Next: 1. Last block: 0s',
            footerIcon: 'ti-dashboard'
          },
          {
            type: 'danger',
            icon: 'ti-bolt',
            title: 'Miner Hashrate',
            value: '0 h/s',
            footerText: 'Stopped',
            footerIcon: 'ti-reload'
          },
          {
            type: 'warning',
            icon: 'ti-pulse',
            title: 'Stored blocks',
            value: '0',
            footerText: 'Nodes: 0. Syncing...',
            footerIcon: 'ti-reload'
          }
        ]
      }
    },
    methods: {
      getInfos: function () {
        astilectron.send({name: 'getInfos'}, (response) => {
          console.log(response)
          const infos = response.payload

          const wallets = infos.wallets

          let pendingAmount = (infos.ownWaitingTx.reduce((memo, item) => memo + item.amount, 0) / 100)
          if (pendingAmount === 0) {
            pendingAmount = ''
          } else {
            pendingAmount = ' (' + ((pendingAmount > 0) ? ('+' + pendingAmount.toFixed(2)) : pendingAmount.toFixed(2)) + ')'
          }

          this.statsCards[0].value = (wallets.reduce((memo, item) => memo + item.amount, 0) / 100).toFixed(2) + pendingAmount
          this.statsCards[0].footerText = wallets.length + ' wallets'

          this.statsCards[1].value = infos.blocksHeight
          this.statsCards[1].footerText = 'Difficulty: ' + infos.difficulty + ', Next: ' + infos.nextDifficulty + '. Last block: ' + infos.timeSinceLastBlock + 's'

          const minerInfo = infos.minerInfo
          this.statsCards[2].value = minerInfo.hashrate + ' h/s'
          this.statsCards[2].footerText = (minerInfo.running ? 'Running' : 'Stopped') + '. Wait tx: ' + minerInfo.waitingTransactions + '. Proc tx: ' + minerInfo.processingTransactions

          this.statsCards[3].value = infos.storedKeys
          this.statsCards[3].footerText = 'Nodes: ' + infos.nodesNb + '. ' + (infos.synced ? 'Synced' : 'Syncing...')
        })
      }
    },
    created () {
      this.getInfos()
      this.timer = setInterval(() => {
        this.getInfos()
      }, 1000)
    },
    destroyed () {
      this.timer.stop()
    }
  }

</script>
<style>

  body, html {
    border: 4px;
  }

</style>
