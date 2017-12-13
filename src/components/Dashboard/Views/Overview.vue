<template>
  <div>

    <div class="row">
      <div class="col-lg-3 col-sm-6" :key="stats.id" v-for="stats in statsCards">
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

    <!--Charts
    <div class="row">

      <div class="col-xs-12">
        <chart-card :chart-data="usersChart.data" :chart-options="usersChart.options">
          <h4 class="title" slot="title">Users behavior</h4>
          <span slot="subTitle"> 24 Hours performance</span>
          <span slot="footer">
            <i class="ti-reload"></i> Updated 3 minutes ago</span>
          <div slot="legend">
            <i class="fa fa-circle text-info"></i> Open
            <i class="fa fa-circle text-danger"></i> Click
            <i class="fa fa-circle text-warning"></i> Click Second Time
          </div>
        </chart-card>
      </div>

      <div class="col-md-6 col-xs-12">
        <chart-card :chart-data="preferencesChart.data"  chart-type="Pie">
          <h4 class="title" slot="title">Email Statistics</h4>
          <span slot="subTitle"> Last campaign performance</span>
          <span slot="footer">
            <i class="ti-timer"></i> Campaign set 2 days ago</span>
          <div slot="legend">
            <i class="fa fa-circle text-info"></i> Open
            <i class="fa fa-circle text-danger"></i> Bounce
            <i class="fa fa-circle text-warning"></i> Unsubscribe
          </div>
        </chart-card>
      </div>
-->
      <div class="col-md-12 col-xs-12">
        <chart-card :chart-data="arrayChart" :chart-options="activityChart.options">
          <h4 class="title" slot="title">Reservas deste ano</h4>
         <!-- <span slot="footer">
            <i class="ti-check"></i> Data information certified</span> -->
          <!-- <div slot="legend">
            <i class="fa fa-circle text-info"></i> Tesla Model S
            <i class="fa fa-circle text-warning"></i> BMW 5 Series 
          </div>-->
        </chart-card>
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
            id: 1,
            type: 'success',
            icon: 'ti-user',
            title: 'Total de Clientes',
            value: '0',
            footerText: 'Atualizado agora',
            footerIcon: 'ti-reload'
          },
          {
            id: 2,
            type: 'success',
            icon: 'ti-calendar',
            title: 'Total de Reservas',
            value: '0',
            footerText: 'Atualizado agora',
            footerIcon: 'ti-reload'
          },
          {
            id: 3,
            type: 'info',
            icon: 'ti-calendar',
            title: 'Reservas desse mês',
            value: '0',
            footerText: 'Atualizado agora',
            footerIcon: 'ti-reload'
          },
          {
            id: 4,
            type: 'info',
            icon: 'ti-user',
            title: 'Clientes desse mês',
            value: '+45',
            footerText: 'Atualizado agora',
            footerIcon: 'ti-reload'
          }
        ],
        usersChart: {
          data: {
            labels: ['9:00AM', '12:00AM', '3:00PM', '6:00PM', '9:00PM', '12:00PM', '3:00AM', '6:00AM'],
            series: [
              [287, 385, 490, 562, 594, 626, 698, 895, 952],
              [67, 152, 193, 240, 387, 435, 535, 642, 744],
              [23, 113, 67, 108, 190, 239, 307, 410, 410]
            ]
          },
          options: {
            low: 0,
            high: 1000,
            showArea: true,
            height: '245px',
            axisX: {
              showGrid: false
            },
            lineSmooth: this.$Chartist.Interpolation.simple({
              divisor: 3
            }),
            showLine: true,
            showPoint: false
          }
        },
        activityChart: {
          data: {
            labels: ['Jan', 'Fev', 'Mar', 'Abr', 'Mai', 'Jun', 'Jul', 'Ago', 'Set', 'Out', 'Nov', 'Dez'],
            // series: [[121,135,125,118,118,147,124,123,124,123,126,133]]
            series: []
          },
          options: {
            seriesBarDistance: 40,
            axisX: {
              showGrid: false
            },
            height: '245px'
          }
        },
        preferencesChart: {
          data: {
            labels: ['62%', '32%', '6%'],
            series: [62, 32, 6]
          },
          options: {}
        }

      }
    },
    created() {
      this.$Progress.start();
      this.$http.get('//websports.herokuapp.com/api/relatorios')
           .then(res => {
             this.statsCards[0].value = res.body.clientes;
             this.statsCards[1].value = res.body.reservas;
             this.statsCards[2].value = res.body.month[new Date().getMonth()].length
             var array = []
             Object.entries(res.body.month).forEach((value,index) => {
               array.push(value[1].length)
             })
             this.activityChart.data.series.push(array)
             this.$emit('reload-chart')
           })
           .catch(err => {
             this.statsCards.map((value, index) => {
               value.type = 'danger'
               value.footerText = 'Erro ao carregar'
               value.footerIcon = 'ti-error'
             })
             console.log(err)
             this.$Progress.fail();
           })
    },
    computed: {
      arrayChart() {
        return this.activityChart.data
      }
    }
  }

</script>
<style>

</style>
