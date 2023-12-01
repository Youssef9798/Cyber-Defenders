<template>
  <div class="container mx-auto">
    <div class="w-full min-h-[calc(100vh-5rem)] bg-white p-10 relative">
      <apexchart
        ref="scatterChart"
        :key="chartKey"
        height="100%"
        type="scatter"
        width="100%"
        :options="options"
        :series="series"
      ></apexchart>
      <transition
        mode="out-in"
        enter-active-class="slide-in-bottom"
        leave-active-class="slide-out-bottom"
      >
        <div
          v-if="showReview"
          class="absolute bg-white rounded-lg shadow-md flex flex-col min-w-[200px] p-4 border border-gray-300 gap-2"
          :style="{
            left: left + 'px',
            top: top + 'px',
          }"
        >
          <div class="flex items-center justify-between gap-2">
            <p class="font-medium text-base text-gray-600">
              {{ certificate?.name }}
            </p>
            <a
              @click="closeReviewModal"
              class="w-7 !cursor-pointer h-7 flex items-center justify-center rounded-full bg-gray-100"
            >
              <close-icon class="w-5 h-5 text-gray-500" />
            </a>
          </div>
          <div
            v-if="!submittedReviews.some((el) => el === certificate.index)"
            class="flex flex-col gap-2"
          >
            <div class="flex flex-wrap gap-3">
              <div
                v-for="(diff, index) in difficulties"
                :key="`${diff.text}-${index}`"
                class="flex align-items-center cursor-pointer p-2 rounded-md transition-all"
                :class="
                  difficulty === diff.value
                    ? 'bg-blue-50 border border-blue-400'
                    : 'bg-transparent border border-transparent'
                "
              >
                <label :for="diff.text" class="capitalize cursor-pointer">
                  <input
                    type="radio"
                    v-model="difficulty"
                    :value="diff.value"
                    name="difficulty"
                    :id="diff.text"
                  />
                  {{ diff.text }}
                </label>
              </div>
            </div>
            <textarea
              cols="3"
              rows="2"
              v-model="additional"
              class="min-w-200 resize-none border border-gray-400 rounded-lg text-sm p-2"
              placeholder="Write additional review (optional)"
            ></textarea>
            <div class="w-full flex items-center justify-end">
              <button
                class="px-4 py-2 text-sm rounded-lg"
                :class="
                  difficulty
                    ? 'bg-[#1e293b] text-white cursor-pointer'
                    : 'bg-gray-100 text-gray-400 cursor-not-allowed select-none'
                "
                :disabled="!difficulty"
                @click="submitReview"
              >
                Submit
              </button>
            </div>
          </div>
          <p v-else class="text-sm font-medium">
            Your review saved successfully
          </p>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import { certs } from '~/data/certs'
import CloseIcon from '~/components/icons/CloseIcon.vue'

export default {
  components: {
    CloseIcon,
  },
  mounted() {
    certs.forEach((cert) => {
      this.series.push({
        name: cert.name,
        data: [[cert.popularity, cert.difficulty]],
      })
    })

    this.$refs.scatterChart?.refresh()
    this.drawAnnotationsQuarters()
  },
  data() {
    return {
      chartKey: 0,
      showReview: false,
      left: 0,
      top: 0,
      difficulty: null,
      difficulties: [
        {
          text: 'beginner',
          value: 1,
        },
        {
          text: 'intermediate',
          value: 2,
        },
        {
          text: 'expert',
          value: 3,
        },
      ],
      additional: null,
      certificate: null,
      submittedReviews: [],
    }
  },
  computed: {
    series() {
      return []
    },
    options() {
      return {
        chart: {
          height: 350,
          type: 'scatter',

          zoom: {
            enabled: false,
          },
          events: {
            dataPointSelection: this.showReviewModal,
          },
        },
        annotations: {
          yaxis: [],
          xaxis: [],
        },
        xaxis: {
          title: {
            text: 'Popularity',
          },
          tickAmount: 10,
          max: 10,
        },
        yaxis: {
          title: {
            text: 'Diffculity',
          },
          tickAmount: 5,
          max: 10,
        },
        dataLabels: {
          enabled: false,
        },
        tooltip: {
          enabled: true,
          custom: function ({ series, seriesIndex, dataPointIndex, w }) {
            var name = w.globals.initialSeries[seriesIndex].name
            var x = w.globals.initialSeries[seriesIndex].data[dataPointIndex][0]
            var y = w.globals.initialSeries[seriesIndex].data[dataPointIndex][1]

            return (
              '<ul style="padding: 10px;">' +
              '<li><b>Name</b>: ' +
              name +
              '</li>' +
              '<li><b>Popularity</b>: ' +
              x +
              '</li>' +
              '<li><b>Diffculity</b>: ' +
              (y >= 0 && y < 3
                ? ' Beginner '
                : y >= 3 && y < 6
                ? ' Intermediate '
                : y >= 6
                ? ' Expert '
                : y) +
              '</li>' +
              '</ul>'
            )
          },
        },
      }
    },
  },
  methods: {
    drawAnnotationsQuarters() {
      const xAxisTick = Math.round(
        this.$refs.scatterChart.chart?.w?.globals?.maxX
      )
      const yAxisTick = Math.round(
        this.$refs.scatterChart.chart?.w?.globals?.maxY
      )
      const gridWidth = Math.round(
        this.$refs.scatterChart.chart?.w?.globals?.gridWidth
      )
      const gridHeight = Math.round(
        this.$refs.scatterChart.chart?.w?.globals?.gridHeight
      )
      const xAxisWidth = Math.round(
        this.$refs.scatterChart.chart?.dimensions?.xAxisWidth
      )
      const yAxisWidth = Math.round(
        this.$refs.scatterChart.chart?.dimensions?.yAxisWidth
      )
      const offsetY = -Math.round(gridHeight / 2)
      const offsetX = Math.round(gridWidth / 2)

      const yAxisAnnoation = {
        y: yAxisTick / 2,
        y2: yAxisTick,
        borderColor: '#000000',
        offsetX: gridWidth / 2,
        offsetY: 0,
        width: '1',
        label: {
          borderColor: '#c2c2c2',
          borderWidth: 1,
          borderRadius: 2,
          text: 'LEADERS',
          offsetX: -(gridWidth / 4 - xAxisTick),
          offsetY: 0,
          style: {
            background: '#ADBAC4',
            color: '#FFFFFF',
            fontSize: '12px',
            fontWeight: 400,
            padding: {
              left: 10,
              right: 10,
              top: 4,
              bottom: 4,
            },
          },
        },
      }
      const yAxisAnnoationTwo = {
        y: yAxisTick / 2,
        y2: yAxisTick,
        borderColor: null,
        fillColor: '#ffffff',
        offsetX: 0,
        offsetY: 0,
        width: '50%',
        label: {
          borderColor: '#c2c2c2',
          borderWidth: 1,
          borderRadius: 2,
          text: 'CHALLENGERS',
          offsetX: -(gridWidth / 2) - gridWidth / 5,
          offsetY: yAxisTick,
          style: {
            background: '#ADBAC4',
            color: '#FFFFFF',
            fontSize: '12px',
            fontWeight: 400,
            padding: {
              left: 10,
              right: 10,
              top: 4,
              bottom: 4,
            },
          },
        },
      }
      const yAxisAnnoationThree = {
        y: 0,
        y2: yAxisTick / 2,
        borderColor: '#000000',
        offsetX: 0,
        offsetY: 0,
        width: '50%',
        label: {
          borderColor: '#c2c2c2',
          borderWidth: 1,
          borderRadius: 2,
          text: 'NICHE PLAYERS',
          offsetX: -(gridWidth / 2) - gridWidth / 5,
          offsetY: gridHeight / 2 - yAxisTick,
          style: {
            background: '#ADBAC4',
            color: '#FFFFFF',
            fontSize: '12px',
            fontWeight: 400,
            padding: {
              left: 10,
              right: 10,
              top: 4,
              bottom: 4,
            },
          },
        },
      }
      const yAxisAnnoationFour = {
        y: 0,
        y2: yAxisTick / 2,
        borderColor: null,
        fillColor: '#ffffff',
        offsetX: gridWidth / 2,
        offsetY: 0,
        width: '1',
        label: {
          borderColor: '#c2c2c2',
          borderWidth: 1,
          borderRadius: 2,
          text: 'VISIONARIES',
          offsetX: -(gridWidth / 4 - xAxisTick),
          offsetY: gridHeight / 2 - yAxisTick,
          style: {
            background: '#ADBAC4',
            color: '#FFFFFF',
            fontSize: '12px',
            fontWeight: 400,
            padding: {
              left: 10,
              right: 10,
              top: 4,
              bottom: 4,
            },
          },
        },
      }
      this.$refs.scatterChart?.addYaxisAnnotation(yAxisAnnoation)
      this.$refs.scatterChart?.addYaxisAnnotation(yAxisAnnoationTwo)
      this.$refs.scatterChart?.addYaxisAnnotation(yAxisAnnoationThree)
      this.$refs.scatterChart?.addYaxisAnnotation(yAxisAnnoationFour)
    },
    closeReviewModal() {
      this.showReview = false
      this.left = null
      this.top = null
      this.difficulty = null
      this.certificate = null
    },
    showReviewModal(event, chartContext, config) {
      this.showReview = true
      this.left = event.offsetX
      this.top = event.offsetY + 60

      this.certificate = config.w.globals.initialSeries[config.seriesIndex]
      this.certificate.index = config.seriesIndex
    },
    submitReview() {
      const seriesRecord = certs[this.certificate.index]
      seriesRecord.popularity += 1
      seriesRecord.difficulty =
        (seriesRecord.difficulty + this.difficulty) / seriesRecord.popularity
      this.submittedReviews.push(this.certificate.index)
      this.$refs.scatterChart?.refresh()
      this.drawAnnotationsQuarters()
      this.closeReviewModal()
      this.$toast.success('Thanks for your review')
    },
  },
}
</script>

<style>
.slide-in-bottom {
  -webkit-animation: slide-in-bottom 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94)
    both;
  animation: slide-in-bottom 0.5s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}
@-webkit-keyframes slide-in-bottom {
  0% {
    -webkit-transform: translateY(100px);
    transform: translateY(100px);
    opacity: 0;
  }
  100% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    opacity: 1;
  }
}
@keyframes slide-in-bottom {
  0% {
    -webkit-transform: translateY(100px);
    transform: translateY(100px);
    opacity: 0;
  }
  100% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    opacity: 1;
  }
}
.slide-out-bottom {
  -webkit-animation: slide-out-bottom 0.5s cubic-bezier(0.55, 0.085, 0.68, 0.53)
    both;
  animation: slide-out-bottom 0.5s cubic-bezier(0.55, 0.085, 0.68, 0.53) both;
}
@-webkit-keyframes slide-out-bottom {
  0% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    opacity: 1;
  }
  100% {
    -webkit-transform: translateY(100px);
    transform: translateY(100px);
    opacity: 0;
  }
}
@keyframes slide-out-bottom {
  0% {
    -webkit-transform: translateY(0);
    transform: translateY(0);
    opacity: 1;
  }
  100% {
    -webkit-transform: translateY(100px);
    transform: translateY(100px);
    opacity: 0;
  }
}
</style>
