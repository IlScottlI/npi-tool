<template>
  <div>
    <v-dialog
      v-model="dialog"
      persistent
    >
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          x-small
          v-bind="attrs"
          icon
          color="blue darken-1"
          v-on="on"
        >
          <v-icon dense>
            mdi-lead-pencil
          </v-icon>
        </v-btn>
      </template>
      <v-card>
        <v-card-title>
          <span class="headline">Details</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-form
              ref="form"
              v-model="valid"
              lazy-validation
              class="compact-form"
            >
              <v-container>
                <h1>NPI Data</h1>
                <v-divider class="mt-5 mb-4" />
                <v-row>
                  <v-col
                    cols="12"
                    sm="3"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.material"
                      label="GCAS"
                      readonly
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    sm="3"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.batchNumber"
                      label="Batch"
                      readonly
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    sm="3"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.buom"
                      label="Cases"
                      readonly
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    sm="3"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.stdPallet"
                      label="Pallets"
                      readonly
                    />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.materialDescription"
                      label="Description"
                      readonly
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    sm="3"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.subSector"
                      label="Business Category"
                      readonly
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    sm="3"
                  >
                    <v-text-field
                      dense
                      :value="cellData.data.materialType"
                      label="Material Type"
                      readonly
                    />
                  </v-col>
                </v-row>
                <v-row>
                  <v-col
                    cols="12"
                    md="6"
                  >
                    <v-textarea
                      v-model="QANotes"
                      dense
                      label="Quality Notes"
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    md="6"
                  >
                    <v-textarea
                      v-model="comments"
                      dense
                      label="Public Notes"
                      counter
                      maxlength="400"
                    />
                  </v-col>
                </v-row>
                <v-spacer />
                <v-row>
                  <v-col
                    cols="12"
                    md="9"
                  >
                    <h1>Work Process</h1>
                  </v-col>
                  <v-col
                    cols="12"
                    md="3"
                  >
                    <v-text-field
                      v-model="legalEntity"
                      dense
                      label="Owner"
                    />
                  </v-col>
                </v-row>

                <v-divider class="mb-5" />
                <v-row>
                  <v-col
                    cols="12"
                    md="3"
                  >
                    <v-select
                      v-model="bucketID"
                      dense
                      :items="buckets"
                      item-text="Name"
                      item-value="ID"
                      label="Bucket"
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    md="3"
                  >
                    <v-select
                      v-if="scrapTypeVis"
                      v-model="scrapType"
                      dense
                      :items="scrapTypes"
                      label="Scrap Type"
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    md="3"
                  />
                  <v-col
                    cols="12"
                    md="3"
                  >
                    <v-menu
                      v-model="modal"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      transition="scale-transition"
                      offset-y
                      min-width="auto"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="dispositionDeadline"
                          label="Disposition Deadline"
                          prepend-icon="mdi-calendar"
                          clearable
                          v-bind="attrs"
                          v-on="on"
                        />
                      </template>
                      <v-date-picker
                        v-model="dispositionDeadline"
                        @input="modal = false"
                      />
                    </v-menu>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col
                    cols="12"
                    md="3"
                  >
                    <v-select
                      v-if="blockClassVis"
                      v-model="blockClass1"
                      dense
                      :items="blockClasses"
                      item-text="name"
                      item-value="id"
                      label="Block Class"
                      required
                    />
                  </v-col>
                  <v-col
                    cols="12"
                    md="3"
                  />
                </v-row>
              </v-container>
            </v-form>
          </v-container>
        </v-card-text>
        <v-card-actions class="mr-16 pb-6">
          <v-spacer />
          <v-btn v-if="loading">
            <v-progress-circular
              :size="30"
              :width="7"
              color="success"
              indeterminate
            />
          </v-btn>
          <v-btn
            v-if="!loading"
            color="blue darken-1"
            text
            :disabled="save"
            class="mr-4"
            @click="updateEvent()"
          >
            Save
          </v-btn>
          <v-btn
            color="blue darken-1"
            text
            @click="dialog = false"
          >
            Cancel
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <toast
      v-if="viewToast"
      :options="toastOptions"
    />
  </div>
</template>

<script>
  import { mapMutations, mapState } from 'vuex'
  import { mdiLeadPencil } from '@mdi/js'
  export default {
    components: {
      Toast: () => import('@/components/core/Toast'),
    },
    props: {
      cellData: {
        type: Object,
        default: () => {},
      },
    },
    data () {
      return {
        toastOptions: { text: 'Update Success', type: 'success', time: 1000 },
        valid: true,
        dialog: false,
        mdiLeadPencil,
        modal: false,
        dispositionDeadline: null,
        bucketID: null,
        QANotes: null,
        blockClassVis: false,
        blockClass1: null,
        comments: null,
        legalEntity: null,
        scrapTypeVis: false,
        scrapType: null,
        viewToast: false,
        save: false,
        loading: false,
      }
    },
    computed: {
      ...mapState('app', ['buckets', 'scrapTypes', 'oldValues']),

      blockClasses () {
        const arr = []
        this.$store.state.npi.npiConfigData.forEach((element) => {
          if (element.dataSource === 'blockClass') {
            arr.push({
              id: '' + element.JSON_Data.id + '',
              name: element.JSON_Data.name,
            })
          }
        })
        return arr
      },
    },
    watch: {
      bucketID (e) {
        this.bucketID = `${e}`
        if (e === this.cellData.data.bucketID) {
          this.blockClassVis = false
        } else {
          this.blockClassVis = true
        }
        if (e === '2') {
          this.scrapTypeVis = true
        } else {
          this.scrapTypeVis = false
        }
      },
      blockClassVis (blockClassVis) {
        if (this.blockClassVis) {
          if (this.blockClass1) {
            this.save = false
          } else {
            this.save = true
          }
        }
      },
      blockClass1 (blockClass1) {
        if (this.bucketID !== this.cellData.data.bucketID) {
          if (blockClass1) {
            if (this.bucketID === '2') {
              if (this.scrapType) {
                this.save = false
              } else {
                this.save = true
              }
            } else {
              this.save = false
            }
          } else {
            this.save = true
          }
        }
      },
      scrapType (scrapType) {
        if (this.bucketID !== this.cellData.data.bucketID) {
          if (scrapType) {
            if (this.bucketID === '2') {
              if (this.blockClass1) {
                this.save = false
              } else {
                this.save = true
              }
            } else {
              this.save = true
            }
          }
        }
      },
    },
    created () {
      if (this.cellData.data.dispositionDeadline) {
        this.dispositionDeadline = this.$moment(
          this.cellData.data.dispositionDeadline,
        ).format('YYYY-MM-DD')
      } else {
        this.dispositionDeadline = this.cellData.data.dispositionDeadline
      }
      this.QANotes = this.cellData.data.QANotes
      this.comments = this.cellData.data.comments
      this.legalEntity = this.cellData.data.legalEntity
      this.bucketID = this.cellData.data.bucketID
      this.scrapType = this.cellData.data.scrapType
      this.blockClass1 = this.cellData.data.blockClass1
    },
    methods: {
      ...mapMutations('app', ['setDialog']),
      validate () {
        this.$refs.form.validate()
      },
      reset () {
        this.$refs.form.reset()
      },
      resetValidation () {
        this.$refs.form.resetValidation()
      },
      updateEvent () {
        this.loading = true
        const body = []
        body.push({
          npiMasterID: this.cellData.data.id,
          updatePrams: {
            dispositionDeadline: this.dispositionDeadline,
            QANotes: this.QANotes,
            comments: this.comments,
            legalEntity: this.legalEntity,
            bucketID: this.bucketID,
            scrapType: this.scrapType,
            blockClass1: this.blockClass1,
          },
          oldValues: this.cellData.data,
          id: this.cellData.data.id,
        })
        fetch('https://tsp-appserverpe.na.pg.com/APIs/NPI/update.php', {
          method: 'POST',
          body: JSON.stringify(body[0]),
        })
          .then((response) => {
            if (response.ok) {
              return response.json()
            } else {
            }
          })
          .then((response) => {
            if (response.response.status === 0) {
              this.viewToast = true
              setTimeout(() => {
                this.viewToast = false
                this.loading = false
              }, 3000)
            } else {
              this.viewToast = false
              this.loading = false
            }
          })
          .catch((err) => {
            console.log(err)
          })
          .finally(() => {
            this.cellData.data.QANotes = this.QANotes
            this.cellData.data.comments = this.comments
            this.cellData.data.bucketID = this.bucketID
            this.cellData.data.scrapType = this.scrapType
            this.cellData.data.blockClass1 = this.blockClass1
            this.cellData.data.dispositionDeadline = this.dispositionDeadline
            this.cellData.data.legalEntity = this.legalEntity
            this.dialog = false
            this.cellData.component.refresh()
          })
      },
    },
  }
</script>

<style>
</style>
