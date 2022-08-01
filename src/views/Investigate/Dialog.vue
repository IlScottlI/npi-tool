<template>
  <v-row justify="center">
    <v-dialog
      v-model="bucketDialog"
      persistent
      width="40%"
    >
      <v-card>
        <v-card-title>
          <span class="headline">Change Bucket</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col
                cols="12"
                sm="6"
              >
                <v-select
                  v-model="selectedItem.data.bucketID"
                  :items="buckets"
                  label="Bucket"
                  item-text="Name"
                  item-value="ID"
                />
              </v-col>
              <v-col
                cols="12"
                sm="6"
              >
                <v-autocomplete
                  v-model="selectedItem.data.blockClass1"
                  :items="blockClasses"
                  label="Block Class *"
                  item-text="name"
                  item-value="id"
                  required
                />
              </v-col>
            </v-row>
          </v-container>
          <small>*indicates required field</small>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="blue darken-1"
            text
            @click="bucketDialog = false"
          >
            Close
          </v-btn>
          <v-btn
            color="blue darken-1"
            text
            @click="bucketDialog = false"
          >
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
  import { mapMutations } from 'vuex'
  export default {
    data () {
      return {
        buckets: [{ ID: '1', Name: 'Investigate' }, { ID: '2', Name: 'Scrap' }, { ID: '3', Name: 'Rework' }, { ID: '4', Name: 'Donate' }, { ID: '5', Name: 'Return to Vendor' }, { ID: '6', Name: 'Release' }],
        scrapTypes: ['', 'Heritage', 'Awesome'],
      }
    },
    computed: {
      bucketDialog: {
        get () {
          return this.$store.state.app.bucketDialog
        },
        set (value) {
          this.toggleBucketDialog(value)
        },
      },
      selectedItem: {
        get () {
          return this.$store.state.app.selectedItem
        },
        set (value) {
          this.setSelectedItem(value)
        },
      },
      blockClasses () {
        const arr = []
        this.$store.state.npi.npiConfigData.forEach(element => {
          if (element.dataSource === 'blockClass') {
            arr.push({ id: '' + element.JSON_Data.id + '', name: element.JSON_Data.name })
          }
        })
        return arr
      },
    },

    methods: {
      ...mapMutations('app', ['toggleBucketDialog']),
      ...mapMutations('app', ['setSelectedItem']),
    },
  }
</script>
