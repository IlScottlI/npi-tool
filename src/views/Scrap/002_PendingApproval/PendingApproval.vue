
<template>
  <div
    v-observe-visibility="{
      callback: visibilityChanged,
      throttle,
      intersection: {
        threshold,
      },
    }"
  >
    <section id="scrap">
      <v-card
        v-if="loading"
        :color="`grey ${theme.isDark ? 'darken-2' : 'lighten-4'}`"
        class="px-3 pt-3 pb-3"
      >
        <v-skeleton-loader
          class="mx-auto"
          type="table-row-divider@10"
        />
      </v-card>
      <v-card v-if="!loading">
        <v-container fluid>
          <div width="100%">
            <dx-data-grid
              id="gridContainer"
              ref="gridContainerInvestigate"
              width="100%"
              overflow="scroll"
              :data-source="npiData"
              :show-column-lines="false"
              :show-row-lines="false"
              :show-borders="false"
              :row-alternation-enabled="true"
              :focused-row-enabled="false"
              :key-expr="'id'"
              :remote-operations="true"
              :allow-column-reordering="true"
              :filter-value="Approval_filterValue"
              :allow-column-resizing="true"
              :on-initialized="onInitialized"
              :repaint-changes-only="true"
              :highlight-changes="true"
              :column-resizing-mode="'widget'"
              :hover-state-enabled="true"
              :column-hiding-enabled="true"
              :column-auto-width="true"
              @exporting="onExporting"
              @row-updating="rowUpdatingEcho"
              @editing-start="editingStartEcho"
              @toolbar-preparing="onToolbarPreparing"
            >
              <dx-paging />
              <dx-pager
                :show-page-size-selector="true"
                :allowed-page-sizes="pageSizes"
                :show-info="true"
              />
              <dx-filter-row :visible="true" />
              <dx-filter-panel :visible="true" />
              <dx-filter-builder-popup />
              <dx-header-filter :visible="true" />
              <dx-scrolling
                column-rendering-mode="virtual"
                :mode="pager"
              />
              <dx-group-panel :visible="true" />
              <dx-grouping :auto-expand-all="true" />
              <dx-export
                :enabled="true"
                :allow-export-selected-data="true"
              />
              <dx-selection
                :allow-select-all="false"
                :deferred="false"
                mode="multiple"
              />
              <dx-state-storing
                :enabled="false"
                type="localStorage"
                storage-key="pending-approval"
              />
              <dx-editing
                :start-edit-action="'dblClick'"
                refresh-mode="repaint"
                mode="batch"
                :allow-updating="true"
              >
                <dx-form>
                  <dx-item
                    :col-count="4"
                    :col-span="2"
                    item-type="group"
                    caption="NPI Data"
                  >
                    <dx-item
                      data-field="material"
                      caption="GCAS"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="batchNumber"
                      caption="Batch"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="buom"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="storageLocation"
                      :disabled="true"
                    />
                    <dx-item
                      :col-span="2"
                      data-field="materialDescription"
                      caption="Description"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="materialType"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="processOrder"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="line"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="hierarchyCategory"
                      :disabled="true"
                    />
                    <dx-item
                      data-field="subSector"
                      :disabled="true"
                    />
                    <dx-item
                      :visible="true"
                      :disabled="true"
                      data-field="actionNote"
                    />
                    <dx-item
                      :col-span="4"
                      :editor-options="{ height: 60 }"
                      data-field="comments"
                      editor-type="dxTextArea"
                    />
                    <dx-item
                      :col-span="4"
                      :col-count="4"
                      item-type="group"
                      caption="Work Process"
                      :visible="true"
                    >
                      <dx-item
                        :col-span="2"
                        :editor-options="{ height: 60 }"
                        data-field="QANotes"
                        editor-type="dxTextArea"
                      >
                        <dx-text-area />
                      </dx-item>
                      <dx-item data-field="dispositionDeadline" />
                      <dx-item data-field="legalEntity" />
                      <dx-item
                        :col-span="4"
                        :col-count="4"
                        item-type="group"
                        :visible="true"
                      >
                        <dx-item data-field="bucketID" />
                        <dx-item
                          data-field="scrapType"
                          :visible="true"
                        >
                          <dx-lookup :data-source="scrapTypes" />
                        </dx-item>
                        <dx-item
                          :visible="true"
                          :col-span="2"
                          data-field="blockClass1"
                        >
                          <dx-lookup
                            value-expr="id"
                            display-expr="name"
                          />
                        </dx-item>
                      </dx-item>
                    </dx-item>
                  </dx-item>
                </dx-form>
              </dx-editing>
              <dx-column-chooser
                v-model="columns"
                :enabled="true"
              />
              <dx-column
                :visible="false"
                data-field="id"
                :allow-editing="false"
              />
              <dx-column
                :visible="false"
                data-field="materialType"
                :allow-editing="false"
              />
              <dx-column
                data-field="hierarchyCategory"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                data-field="subSector"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                caption="NPI #"
                data-field="npiNumber"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                caption="Blocked Date"
                data-field="Created"
                data-type="date"
                :allow-editing="false"
                :width="96"
                :visible="false"
              />
              <dx-column
                caption="Days Blocked"
                data-field="days_pending"
                cell-template="DaysCell"
                data-type="number"
                :allow-editing="false"
                :width="95"
              />
              <dx-column
                caption="Days Pending Execution"
                data-field="days_in_bucket"
                cell-template="DaysCell"
                data-type="number"
                :allow-editing="false"
                :width="95"
              />
              <dx-column
                caption="Days Pending Steam Doc Approval"
                data-field="days_after_submittion"
                cell-template="DaysCell"
                data-type="number"
                :allow-editing="false"
                :width="95"
              />
              <dx-column
                data-field="line"
                caption="Line"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                caption="Steam #"
                data-field="steamNumber"
                cell-template="SteamCell"
                :visible="true"
              />
              <dx-column
                data-field="steamStatus"
                :visible="true"
              />
              <dx-column
                data-field="processOrder"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                data-field="material"
                :allow-editing="false"
              />
              <dx-column
                data-field="steamNumber_"
                caption="steam"
                :allow-editing="false"
                :visible="false"
              />
              <dx-column
                data-field="steamStatus_"
                caption="status"
                :allow-editing="false"
                :visible="false"
              />
              <dx-column
                data-field="batchNumber"
                caption="Batch"
                :allow-editing="false"
              />
              <dx-column
                data-field="materialDescription"
                caption="Description"
                width="244"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                data-field="buom"
                :allow-editing="false"
                caption="Buom Stock"
                :visible="true"
              />
              <dx-column
                data-field="buomName"
                caption="UOM"
                :allow-editing="false"
                :visible="false"
              />
              <dx-column
                data-field="storageLocation"
                caption="Loc"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                data-field="stdPallet"
                caption="Pallets"
                :visible="true"
                :allow-editing="false"
              />
              <dx-column
                data-field="localStock"
                format="currency"
                caption="Local Stock"
                :visible="true"
                :allow-editing="false"
              />
              <dx-column
                data-field="scrapType"
                :allow-clearing="true"
              >
                <dx-lookup :data-source="scrapTypes" />
              </dx-column>
              <dx-column
                data-field="QANotes"
                caption="QA Notes"
                :visible="false"
              />

              <dx-column
                data-field="blockClassName"
                caption="Block Class"
                :visible="false"
              />
              <dx-column
                data-field="jsonSteamNumber"
                caption="Steam #"
                :visible="false"
              />
              <dx-column
                :visible="false"
                data-field="blockClass1"
              >
                <dx-lookup
                  :data-source="blockClasses"
                  value-expr="id"
                  display-expr="name"
                />
              </dx-column>
              <dx-column
                data-field="nextAction"
                caption="Next Action"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                data-field="inCategoryDateCounter"
                caption="Pending Scrap Action"
                :visible="false"
                :allow-editing="false"
              />
              <dx-column
                data-field="actionNote"
                :visible="false"
              />
              <dx-column
                data-field="comments"
                caption="Public notes"
                :visible="true"
                :width="253"
              />
              <dx-column
                data-field="dispositionDeadline"
                data-type="date"
                :visible="true"
              />
              <dx-column
                data-field="legalEntity"
                caption="Owner"
                :visible="true"
              />
              <dx-column
                data-field="npiType"
                :visible="false"
              >
                <dx-lookup
                  :data-source="npiBuckets"
                  value-expr="id"
                  display-expr="bucket"
                />
              </dx-column>
              >
              <dx-column
                data-field="bucketID"
                caption="Bucket"
                :visible="false"
              >
                <dx-lookup
                  :data-source="buckets"
                  value-expr="ID"
                  display-expr="Name"
                />
              </dx-column>
              <dx-column
                cell-template="EditCell"
                caption=""
                :fixed="true"
                :width="50"
                fixed-position="right"
              />
              <dx-column
                data-field="swifferCategory"
                :visible="false"
              />
              <template #EditCell="{ data: cellData }">
                <edit-form :cell-data="cellData" />
              </template>
              <template #DaysCell="{ data: cellData }">
                <days-cell :cell-data="cellData" />
              </template>
              <template #SteamCell="{ data: cellData }">
                <steam-cell :cell-data="cellData" />
              </template>
            </dx-data-grid>
          </div>
        </v-container>
      </v-card>
      <toast
        v-if="viewToast"
        :options="toastOptions"
      />
    </section>
  </div>
</template>

<script>
  import {
    DxDataGrid,
    DxColumn,
    DxExport,
    DxGroupPanel,
    DxScrolling,
    DxSelection,
    DxGrouping,
    DxEditing,
    DxColumnChooser,
    DxHeaderFilter,
    DxFilterRow,
    DxFilterPanel,
    DxFilterBuilderPopup,
    DxPager,
    DxPaging,
    DxLookup,
    DxForm,
    DxStateStoring,
  } from 'devextreme-vue/data-grid'
  import { exportDataGrid } from 'devextreme/excel_exporter'
  import { DxTextArea } from 'devextreme-vue/text-area'
  import ExcelJS from 'exceljs'
  import saveAs from 'file-saver'
  import { mapActions, mapState, mapGetters, mapMutations } from 'vuex'
  import SteamCell from './SteamCell.vue'
  import DaysCell from '../../../components/core/Days'
  import { DxItem } from 'devextreme-vue/form'
  import query from 'devextreme/data/query'
  const MILLISECONDS_IN_DAY = 1000 * 60 * 60 * 24
  export default {
    name: 'HomeGrid',
    inject: ['theme'],
    components: {
      SteamCell,
      DxDataGrid,
      DxStateStoring,
      DxColumn,
      DxTextArea,
      DxExport,
      DxGroupPanel,
      DxGrouping,
      DxScrolling,
      DxSelection,
      DxColumnChooser,
      DxItem,
      DxEditing,
      DxForm,
      DxHeaderFilter,
      DxFilterRow,
      DxFilterPanel,
      DxFilterBuilderPopup,
      DxPager,
      DxPaging,
      DxLookup,
      DaysCell,
      Toast: () => import('@/components/core/Toast'),
      EditForm: () => import('./EditForm'),
    },

    data () {
      return {
        popupVisible: false,
        isPhoneVisible: true,
        Quality_Notes: null,
        QualityNoteOptions: {
          height: 100,
        },
        Public_Notes: null,
        publicNoteOptions: {
          height: 100,
        },
        Owner: null,
        SteamNumber: null,
        dispositionDeadline: null,
        showScrapType: false,
        selectedBucketId: '0',
        isScrapTypeVisible: false,
        isFormDisabled: false,
        bucketOptions: {
          onValueChanged: (e) => {
            this.watchBucket = e.value
          },
          value: '1',
        },
        selected: null,
        editForm: false,
        viewToast: false,
        watchBucket: '1',
        changes: {},
        toastOptions: { text: 'Update Success', type: 'success', time: 1000 },
        buckets: [
          { ID: '1', Name: 'Investigate' },
          { ID: '2', Name: 'Scrap' },
          { ID: '3', Name: 'Rework' },
          { ID: '4', Name: 'Donate' },
          { ID: '5', Name: 'Return to Vendor' },
          { ID: '6', Name: 'Release' },
          { ID: '7', Name: 'Unpickable' },
        ],
        scrapTypes: ['', 'Heritage', 'Awesome'],
        Create_pager: 'virtual',
        pageSizes: [5, 10, 20, 100],
        vis: false,
        file: null,
        selectedSheet: null,
        sheetName: null,
        collection: [],
        selectedRows: [],
        selectedRowKeys: [],
        columns: null,
        steamNumber: null,
        filterBuilderPopupPosition: {
          of: window,
          at: 'top',
          my: 'top',
          offset: { y: 10 },
        },
        editButtons: ['edit'],
        showFilterRow: true,
        events: [],
        welcomeMsg: '',
        showSteamButton: false,
        dataGrid: null,
        show: true,
        isVisible: true,
        throttle: 0,
        threshold: 0,
        loading: false,
      }
    },
    computed: {
      ...mapState('npi', [
        'npiConfigData',
        'Category',
        'Material',
        'npiBuckets',
        'CategoryBucket',
      ]),
      ...mapGetters('npi', [
        'rootCauseData',
        'rootCauseData',
        'filteredData',
        'token',
      ]),
      pager: {
        get () {
          return this.$store.state.app.pager
        },
        set (value) {
          this.setPager(value)
        },
      },
      oldValues: {
        get () {
          return this.$store.state.app.oldValues
        },
        set (value) {
          this.setOldValues(value)
        },
      },
      activeBtn () {
        return this.selectedRowKeys.length === 1
      },
      selectedItem: {
        get () {
          return this.$store.state.app.selectedItem
        },
        set (value) {
          this.setSelectedItem(value)
          this.dialogCLose1 = null
        },
      },
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
      bucketDialog: {
        get () {
          return this.$store.state.app.bucketDialog
        },
        set (value) {
          this.toggleBucketDialog(value)
        },
      },
      selectedRowObject: {
        get () {
          return this.$store.state.app.selectedRowObject
        },
        set (value) {
          this.setSelectedRowObject(value)
        },
      },
      selectedRowsData: {
        get () {
          return this.$store.state.app.selectedRowsData
        },
        set (value) {
          this.setSelectedRowsData(value)
        },
      },
      dialog: {
        get () {
          return this.$store.state.app.dialog
        },
        set (value) {
          this.setDialog(value)
        },
      },
      sheets () {
        return [{ name: 'Steam Export', data: this.collection }]
      },
      collection_ () {
        const result = []
        try {
          this.collection.forEach((element) => {
            result.push({
              Category: element.hierarchyCategory,
              Material_Type: element.materialType,
              Material: element.material,
              Batch: element.batchNumber,
              BUOM_Stock: element.buom,
              Stock_$: element.localStock,
              Action_Note: element.actionNote,
              Public_Notes: element.comments,
              Root_Cause: element.rootCauseName,
              Stop_Ship_Date: element.stopShipDate_,
              Batch_Exp_Date: element.batchExpiryDate_,
              TDCVAL: element.tdcVal,
              Plant: element.plant,
            })
          })
        } catch (error) {}
        return result
      },
      Approval_filterValue () {
        const response = []
        response.push(['steamNumber', 'noneof', ['']])
        response.push('and')
        response.push(['steamStatus_', 'noneof', ['Approved']])
        response.push('and')
        response.push(['bucketID', '=', '2'])
        if (this.Category) {
          response.push('and')
          response.push(['hierarchyCategory', 'anyof', [this.Category]])
        }
        if (this.Material) {
          response.push('and')
          response.push(['materialType', 'anyof', [this.Material]])
        }
        if (this.CategoryBucket) {
          response.push('and')
          response.push(['npiType', 'anyof', [this.CategoryBucket]])
        }
        return response
      },
      groupedBy: {
        get () {
          return this.$store.state.npi.groupedBy
        },
        set (value) {
          this.setgroupedBy(value)
        },
      },
      npiMaster: {
        get () {
          return this.$store.state.npi.npiMaster
        },
        set (value) {
          this.setSelectedChunk(value)
        },
      },
      npiData: {
        get () {
          return this.$store.state.npi.npiData
        },
        set (value) {
          this.setNPIData(value)
        },
      },
    },

    watch: {
      watchBucket (e) {
        if (e === '2') {
          this.showScrapType = true
        } else {
          this.showScrapType = false
        }
        this.bucketOptions.value = e
      },
      groupedBy (item) {
        this.options.groupBy = [item]
      },
      bucketDialog (bucketDialog) {
        if (bucketDialog !== true) {
          this.selected = null
        // this.dialogCLose1 = false
        }
      },
    },

    methods: {
      ...mapMutations('npi', ['setgroupedBy', 'setNPIData']),
      ...mapMutations('app', [
        'toggleBucketDialog',
        'setSelectedItem',
        'setDialog',
        'setSelectedRowsData',
        'setSelectedRowObject',
        'setOldValues',
        'setPendingApprovalGrid',
      ]),
      ...mapActions('npi', ['getNPIData', 'updateNPI', 'getNPIMaster']),
      visibilityChanged: function (isVisible, entry) {
        this.isVisible = isVisible
        if (isVisible) {
          setTimeout(() => {
            this.dataGrid.refresh()
            this.dataGrid.repaint()
          }, 400)
        }
      },
      onToolbarPreparing (e) {
        const dataGrid = e.component
        const toolbarItems = e.toolbarOptions.items
        // Adds a new item
        // toolbarItems.push({
        //   widget: 'dxButton',
        //   options: {
        //     icon: 'clearformat',
        //     onClick: function () {
        //       localStorage.removeItem('pending-approval')
        //       window.location.reload()
        //     },
        //     hint: 'Clear Saved Filter',
        //   },
        //   location: 'after',
        // })
        toolbarItems.push({
          widget: 'dxButton',
          options: {
            icon: 'refresh',
            onClick: function () {
              dataGrid.refresh()
            },
            hint: 'Reload Table',
          },
          location: 'after',
        })
      },
      clearSaved (item) {
        localStorage.removeItem(item)
        window.location.reload()
      },
      calculateStatistics () {
        this.dataGrid.getSelectedRowsData().then((rowData) => {
          var commonDuration = 0

          for (var i = 0; i < rowData.length; i++) {
            commonDuration +=
              rowData[i].Task_Due_Date - rowData[i].Task_Start_Date
          }
          commonDuration = commonDuration / MILLISECONDS_IN_DAY
          this.taskCount = rowData.length
          this.peopleCount = query(rowData)
            .groupBy('ResponsibleEmployee.Employee_Full_Name')
            .toArray().length
          this.avgDuration = Math.round(commonDuration / rowData.length) || 0
        })
      },

      validateForm (e) {
        e.component.validate()
        console.log(e.component)
      },
      handlePropertyChange: function (e) {
        console.log(e)
        if (e.name === 'changedProperty') {
        // handle the property change here
        }
      },
      checkScrap (e) {
        console.log(e)
      },
      echo (e) {
        console.log(e)
      },
      selectionChangedEcho (e) {
        console.log('selectionChanged', e)
        this.selectedRowKeys = e.selectedRowKeys
        this.selectedRowsData = e.selectedRowsData
        this.oldValues = e.selectedRowsData[0]
        if (this.selectedRowsData.length === 1) {
          this.setSelectedRowObject(e.selectedRowsData[0])
        } else {
          this.selectedRowObject = {}
        }
      },
      rowUpdatingEcho (e) {
        console.log('rowUpdating', e)
        this.updateEvent(e)
      },
      editingStartEcho (e) {
        console.log('editingStart', e)
      },
      optionChangedEcho (e) {
        console.log('optionChanged', e)
      },
      onInitialized (e) {
        this.dataGrid = e.component
        this.setPendingApprovalGrid(e.component)
      },
      onSelectionChanged () {},
      startEditAction (e) {
        console.log('startEditAction', e)
      },
      updateSteamNumbers () {
        console.log(this.selectedRows.length)
        this.selectedRows.forEach((element) => {
          const event = {
            steamUpdate: true,
            newData: { steamNumber: this.steamNumber },
            oldData: element,
          }
          this.updateEvent(event)
        })
        this.getNPIData()
        this.getNPIMaster()
      },
      updateEvent (event) {
        console.log(event)
        if (event.oldData.npiNumber) {
        } else {
          event.oldData.npiNumber = event.oldData.id
        }
        if (event.newData.actionNote) {
        } else {
          event.newData.actionNote = 'Scrap'
        }
        console.log(event.oldData.npiNumber)
        const body = []
        body.push({
          steamUpdate: event.steamUpdate,
          npiMasterID: event.oldData.npiMasterID,
          updatePrams: event.newData,
          oldValues: event.oldData,
          id: event.oldData.npiNumber,
          Cookie: this.token,
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
              this.dataGrid.refresh()
              this.dataGrid.clearSelection()
              this.$store.state.app.pendingSteamGrid.refresh()
              this.$store.state.app.pendingApprovalGrid.refresh()
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
            this.dataGrid.refresh()
            this.dataGrid.clearSelection()
            this.$store.state.app.pendingSteamGrid.refresh()
            this.$store.state.app.pendingApprovalGrid.refresh()
          })
      },
      onChange (event) {
        this.file = event.target.files ? event.target.files[0] : null
      },
      addSheet () {
        this.sheets.push({ name: this.sheetName, data: [...this.collection] })
        this.sheetName = null
      },
      peAction (rowData) {
        let item
        if (rowData.peAction === true) {
          item = true
        } else {
          item = false
        }
        return item
      },
      onExporting (e) {
        const workbook = new ExcelJS.Workbook()
        const worksheet = workbook.addWorksheet('NPI Tool')
        exportDataGrid({
          component: e.component,
          worksheet: worksheet,
          autoFilterEnabled: false,
        }).then(function () {
          // https://github.com/exceljs/exceljs#writing-xlsx
          workbook.xlsx.writeBuffer().then(function (buffer) {
            saveAs(
              new Blob([buffer], { type: 'application/octet-stream' }),
              'Investigate.xlsx',
            )
          })
        })
        e.cancel = true
      },
      getRootCause (item, configData) {
        let response = item
        try {
          configData.forEach((element) => {
            if (item === element.JSON_Data.id) {
              response = element.JSON_Data.name
            }
          })
        } catch (error) {}

        return response
      },
      save (item) {
        this.updateNPI(item)
        this.snack = true
        this.snackColor = 'success'
        this.snackText = 'Data saved'
      },
      cancel () {
        this.snack = true
        this.snackColor = 'error'
        this.snackText = 'Canceled'
      },
      open () {
        this.snack = true
        this.snackColor = 'info'
        this.snackText = 'Dialog opened'
      },
      close () {
        console.log('Dialog closed')
      },
    },
  }
</script>
