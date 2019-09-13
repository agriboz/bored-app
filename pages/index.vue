<template>
  <div class="container">
    <b-card no-body>
      <b-tabs card>
        <b-tab title="Activities" active>
          <b-card>
            <div class="row">
              <div class="col d-flex flex-column">
                <b-card
                  :bg-variant="item.error && 'danger'"
                  :text-variant="item.error && 'white'"
                  tag="div"
                  class="mb-2 flex-fill"
                >
                  <b-card-text>
                    {{ item.activity }}
                  </b-card-text>
                  <b-card-text v-if="item.error">
                    {{ item.error }}
                  </b-card-text>
                </b-card>
                <b-alert v-if="notification" show dismissible>
                  {{ notification }}
                </b-alert>
                <b-alert
                  :show="dismissCountDown"
                  dismissible
                  fade
                  variant="success"
                  @dismiss-count-down="countDownChanged"
                >
                  Saved Successfulty
                </b-alert>
                <div v-if="!item.error" class="d-flex justify-content-center">
                  <button class="btn btn-primary" @click="saveForLater">
                    Save For Later
                  </button>
                </div>
              </div>
              <div class="col">
                <div class="form-group">
                  <label for="type">Activity Details:</label>
                  <select id="type" v-model="filter.type" class="form-control">
                    <option v-for="(t, i) in activityTypes" :key="i" :value="t">
                      {{ t }}</option
                    >
                  </select>
                </div>
                <div class="form-group">
                  <label for="participants">Participants:</label>
                  <input
                    id="participants"
                    v-model.number="filter.participants"
                    type="number"
                    min="1"
                    max="666"
                    class="form-control"
                    placeholder="Participants"
                  />
                </div>
                <div class="form-group">
                  <label for="budget">Budget</label>
                  <b-form-input
                    id="bugdet"
                    v-model.number="filter.maxprice"
                    type="range"
                    min="0"
                    max="1"
                    step="1"
                  ></b-form-input>
                  <p class="d-flex justify-content-between">
                    <span>cheap</span>
                    <span>expensive</span>
                  </p>
                </div>
                <button
                  class="btn btn-primary btn-block"
                  @click="getRandomActivity()"
                >
                  Hit me with a new one
                </button>
              </div>
            </div>
          </b-card>
        </b-tab>
        <b-tab title="My List" @click="getActivitiesByKey">
          <b-card-text>
            <b-list-group>
              <b-list-group-item v-for="(s, i) in savedActivities" :key="i">
                <div class="d-flex w-100 justify-content-between">
                  <h5 class="mb-1">{{ capitalize(s.type) }}</h5>
                  <button class="btn btn-danger" @click="removeActivity(s.key)">
                    Delete
                  </button>
                </div>

                <p class="mb-1">
                  {{ s.activity }}
                </p>
                <small class="text-uppercase">
                  Participants {{ s.participants }}</small
                >
              </b-list-group-item>
            </b-list-group>
            <b-alert v-if="!savedActivities.length" show>
              Do you want to add any activities?
            </b-alert>
            <button
              v-if="savedActivities.length"
              class="btn btn-block btn-danger mt-3"
              @click="clearAll"
            >
              Clear All
            </button>
          </b-card-text>
        </b-tab>
      </b-tabs>
    </b-card>
  </div>
</template>

<script>
export default {
  data: () => ({
    savedActivities: [],
    notification: null,
    filter: {
      maxprice: 0,
      type: 'Social',
      participants: 1
    },
    item: {
      activity: null,
      type: null,
      participants: null
    },
    activityTypes: [
      'Education',
      'Recreational',
      'Social',
      'Diy',
      'Charity',
      'Cooking',
      'Relaxation',
      'Music',
      'Busywork'
    ],
    dismissSecs: 2,
    dismissCountDown: 0,
    showDismissibleAlert: false
  }),

  beforeMount() {
    this.initRandomActivity()
    this.getActivitiesByKey()
  },

  methods: {
    countDownChanged(dismissCountDown) {
      this.dismissCountDown = dismissCountDown
    },
    showAlert() {
      this.dismissCountDown = this.dismissSecs
    },
    capitalize(s) {
      if (typeof s !== 'string') return ''
      return s.charAt(0).toUpperCase() + s.slice(1)
    },
    saveForLater() {
      const getKeys = () => {
        return localStorage.getItem('keys')
          ? JSON.parse(localStorage.getItem('keys'))
          : []
      }
      const setKeys = (data) =>
        localStorage.setItem('keys', JSON.stringify(data))

      if (getKeys().includes(this.item.key)) {
        this.notification = 'You have already saved for later'
      }

      if (!getKeys().includes(this.item.key)) {
        setKeys([...getKeys(), this.item.key])
        this.showAlert()
        this.notification = null
      }
    },

    async initRandomActivity() {
      const { data } = await this.$axios.get(`activity`)
      this.item = data
    },

    async getRandomActivity() {
      const { type, participants, maxprice } = this.filter

      const { data } = await this.$axios.get(
        `activity?${type}&participants=${participants}&maxprice=${maxprice}`
      )
      this.item = data
    },

    removeActivity(currentKey) {
      this.savedActivities = this.savedActivities.filter(
        (item) => item.key !== currentKey
      )

      const getKeys = () => {
        return localStorage.getItem('keys')
          ? JSON.parse(localStorage.getItem('keys'))
          : []
      }
      const setKeys = (data) =>
        localStorage.setItem('keys', JSON.stringify(data))

      setKeys(getKeys().filter((key) => key !== currentKey))
    },

    clearAll() {
      localStorage.clear()
      this.savedActivities = []
    },

    getActivitiesByKey() {
      const getActivity = (params = {}) => {
        return this.$axios.get(`activity`, {
          params
        })
      }
      const getKeys = () => {
        return localStorage.getItem('keys')
          ? JSON.parse(localStorage.getItem('keys'))
          : []
      }
      const keys = getKeys()

      return Promise.all(keys.map((key) => getActivity({ key }))).then(
        (responses) => {
          this.savedActivities = responses.map((response) => response.data)
        }
      )
    }
  }
}
</script>
