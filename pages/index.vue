<template>
  <div class="container">
    <b-card no-body>
      <b-tabs card>
        <b-tab title="Activities" active>
          <b-card>
            <div class="row">
              <div class="col">
                <div class="form-group">
                  <label for="activity">You should:</label>
                  <input
                    id="activity"
                    v-model="item.activity"
                    type="text"
                    class="form-control"
                    placeholder="Activity"
                  />
                </div>
              </div>
              <div class="col">
                <div class="form-group">
                  <label for="type">Activity Details:</label>
                  <select id="type" v-model="item.type" class="form-control">
                    <option v-for="(t, i) in activityTypes" :key="i" :value="t">
                      {{ t }}</option
                    >
                  </select>
                </div>
                <div class="form-group">
                  <label for="participants">Participants:</label>
                  <input
                    id="participants"
                    v-model="item.participants"
                    type="text"
                    class="form-control"
                    placeholder="Participants"
                  />
                </div>
                <div class="form-group">
                  <label for="budget">Budget</label>
                  <b-form-input
                    id="bugdet"
                    v-model="item.price"
                    step="0.15"
                    type="range"
                    min="0"
                    max="1"
                  ></b-form-input>
                  <div class="mt-2">Value: {{ value }}</div>
                </div>
                <button class="btn btn-primary btn-block">
                  Hit me with a new one
                </button>
              </div>
            </div>
          </b-card>
        </b-tab>
        <b-tab title="My List">
          <b-card-text>Tab Contents 2</b-card-text>
        </b-tab>
      </b-tabs>
    </b-card>
  </div>
</template>

<script>
export default {
  data: () => ({
    item: {
      activity: null,
      type: null,
      participants: null
    },
    activityTypes: [
      'education',
      'recreational',
      'social',
      'diy',
      'charity',
      'cooking',
      'relaxation',
      'music',
      'busywork'
    ]
  }),

  beforeMount() {
    this.initRandomActivity()
  },

  methods: {
    async initRandomActivity() {
      const { data } = await this.$axios.get(`activity`)
      this.item = data
    },

    async getRandomActivity() {
      // const { data } = await this.$axios.get(`activity`)
      // this.item = data
    }
  }
}
</script>
