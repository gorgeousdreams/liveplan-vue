<template>
  <div class="viewAsset bg-light">
    <div class="main-gradient">
      <div class="title-container mx-auto">
        <b-breadcrumb :items="items" class="p-0"/>
        <div class="d-flex flex-column flex-md-row justify-content-between align-items-center">
          <h1 class="text-white text-left">{{asset.name}} </h1>
          <button class='btn btn-sm icon-btn text-regular' @click="deleteAsset()">
            <i class="flaticon stroke trash-2 text-danger"></i>
            Delete Asset
          </button>
        </div>
      </div>
      <!-- chart Wrapper -->
      <div class="chart-container">
        <barchart v-if="plan.asset_amounts && plan.asset_amounts.length && planStartYear && planEndYear"
                  :label="asset.name"
                  :dataObject="plan.asset_amounts"
                  :startYear="plan.start_year"
                  :endYear="plan.end_year"
                  :planStartYear="planStartYear"
                  :planEndYear="planEndYear"
                  :birthYear="1981"></barchart>
      </div>
    </div>
    <div class="card-container">
      <div class="bg-light mx-xl-auto">
        <!-- Auto Calculation, Custom Values Tabs Card-->
        <b-card no-body class="asset-tabs-card depth-1">
          <b-tabs pills card v-model="tabIndex">
            <b-tab title="Auto Calculation" :title-link-class="'asset-tab'">
              <div class="custom-values-bar" v-if="!customDisabled">
                <b-alert show variant="warning" class="d-flex justify-content-center align-items-center">Values are disabled since asset is in custom mode</b-alert>
              </div>
              <b-form id="autoCalcForm" ref="autoCalcForm" @change="autoCalcformChanged" novalidate :validated="autoCalcValidated" @submit.prevent="onAutoCalcSumbit" v-bind:class="{ 'custom-is-enabled': !customDisabled }">
                <b-container fluid>
                  <b-row class="text-left mx-auto">
                    <b-col md="6" xl="4">
                      <span class="text-regular">Period (start - end year)</span>
                      <div class="d-flex element-spacer">
                        <b-form-group label-for="startYearInput" :invalid-feedback="invalidStartYearFeedback" :state="startYearState">
                          <b-form-input id="startYearInput" required v-model="asset.start_year" type="number" size="sm" v-b-tooltip.hover.bottom title="The year when this asset first occurs." class="text-regular start_end_year start_year" style="width: 86px" placeholder="Start year" :min="planStartYear" :max="asset.end_year - 1" :disabled="!customDisabled"></b-form-input>
                        </b-form-group>
                        <span class="date-spacer">-</span>
                        <b-form-group label-for="endYearInput" :invalid-feedback="invalidEndYearFeedback" :state="endYearState">
                          <b-form-input id="endYearInput" required v-model="asset.end_year" type="number" size="sm" v-b-tooltip.hover.bottom title="The year when this asset stops occuring." class="text-regular start_end_year end_year" style="width: 86px" placeholder="End year" :min="planStartYear + 1" :max="planEndYear" :disabled="!customDisabled"></b-form-input>
                        </b-form-group>
                      </div>
                    </b-col>
                    <b-col md="6" xl="4">
                      <span class="text-regular">Annual Growth Rate</span>
                      <div class="d-flex mb-3">
                        <vue-numeric currency="%" currency-symbol-position="suffix" v-model="asset.annual_growth_rate"  class="form-control form-control-sm element-spacer text-regular annual-growth-rate" :min="0" :max="10" :disabled="!customDisabled"></vue-numeric>
                        <b-input-group size="sm" class="element-spacer">
                          <b-form-input v-model="asset.annual_growth_rate" min="0" max="10" class="slider" type="range" :disabled="!customDisabled"></b-form-input>
                        </b-input-group>
                      </div>
                    </b-col>
                    <b-col md="6" xl="4">
                      <span class="text-regular">Amount</span>
                      <div class="d-flex mb-3">
                        <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="asset.amount" class="form-control form-control-sm element-spacer text-regular amount" :min="0" :disabled="!customDisabled"></vue-numeric>
                        <b-input-group size="sm" class="element-spacer">
                          <b-form-input v-model="asset.amount" min="0" max="100000" step="500" class="slider" type="range" :disabled="!customDisabled"></b-form-input>
                        </b-input-group>
                      </div>
                    </b-col>
                    <b-col md="6" xl="4">
                      <span class="text-regular">Deposit Amount</span>
                      <div class="d-flex mb-3">
                        <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="asset.deposit_amount" class="form-control form-control-sm element-spacer text-regular amount" :min="0" :disabled="!customDisabled"></vue-numeric>
                        <b-input-group size="sm" class="element-spacer">
                          <b-form-input v-model="asset.deposit_amount" min="0" max="50000" step="1000" class="slider" type="range" :disabled="!customDisabled"></b-form-input>
                        </b-input-group>
                      </div>
                    </b-col>
                    <b-col md="6" xl="4">
                      <span class="text-regular">Withdrawal Amount</span>
                      <div class="d-flex mb-3">
                        <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="asset.withdrawal_amount" class="form-control form-control-sm element-spacer text-regular amount" :min="0" :disabled="!customDisabled"></vue-numeric>
                        <b-input-group size="sm" class="element-spacer">
                          <b-form-input v-model="asset.withdrawal_amount" min="0" max="50000" step="1000" class="slider" type="range" :disabled="!customDisabled"></b-form-input>
                        </b-input-group>
                      </div>
                    </b-col>
                  </b-row>
                </b-container>
                <b-button type="submit" :size="'sm'" variant="primary" class="save-calc-btn mx-auto" :disabled="isCalcSaveDisabled || !customDisabled"  v-b-tooltip.hover.bottom title="You can save any changes for this asset by clicking here.">
                  <i class="fa fa-spinner fa-spin" v-if="isSaving" style="margin-top: 2px" ></i>
                  <i class="flaticon stroke checkmark" v-else></i>
                  Save New Values
                </b-button>
              </b-form>
            </b-tab>
            <b-tab title="Custom Values" :title-link-class="'asset-tab'">
              <div class="upload-file-btn">
                <button class='btn btn-sm icon-btn text-regular' :disabled="customDisabled">
                  <i class="flaticon stroke upload text-primary"></i>
                  Upload Files
                </button>
              </div>
              <div class="custom-values-bar">
                <div class="d-flex flex-column flex-lg-row justify-content-lg-between align-items-lg-center" :class="[customDisabled ? 'bar-disable-color' :'bar-enable-color']">
                  <div class="d-flex align-items-center">
                    <span class="mr-3" :class="{ 'font-weight-bold': !customDisabled }">Enable</span>
                    <switches v-model="customDisabled" theme="bootstrap" type-bold="true" color="success"></switches>
                    <span class="ml-3" :class="{ 'font-weight-bold': customDisabled }">Disable</span>
                  </div>
                  <span v-if="customDisabled" class="text-gray text-left label-text">
                    Enabling "Custom Values" will disable "Auto Calculation". You can choose one of the two.
                  </span>
                  <span v-else class="text-gray text-left label-text">
                    Enabling "Auto Calculation" will disable "Custom Values". You need to choose one of the two.
                  </span>
                </div>
              </div>
              <div class="table-container text-regular text-left">
                <b-table show-empty
                        stacked="md"
                        :items="asset.asset_amounts"
                        :fields="fields"
                        :current-page="currentPage"
                        :per-page="perPage"
                        :filter="filter"
                        :sort-by.sync="sortBy"
                        :sort-desc.sync="sortDesc"
                        :sort-direction="sortDirection"
                        @filtered="onFiltered"
                        hover
                        fixed
                >
                  <template slot="top-row" slot-scope="data">
                    <td colspan="3"><b-form-input v-model="filter" size="sm" placeholder="Type to Search" /></td>
                  </template>
                  <template slot="amount" slot-scope="row">
                    <vue-numeric v-show="row.item.is_edit"  @keyup.native.enter.stop.prevent="saveRow(row.item)" currency="SEK" currency-symbol-position="suffix" thousand-separator=" "  v-model="row.item.edit_amount" class="form-control form-control-sm text-regular amount" :minus="true"></vue-numeric>
                    <span v-show="!row.item.is_edit"  @dblclick="editRow(row.item)">{{row.item.amount ? row.item.amount.toLocaleString('sv-SE') : 0}} SEK </span>
                  </template>
                  <template slot="deposit_amount" slot-scope="row">
                    <vue-numeric v-show="row.item.is_edit"  @keyup.native.enter.stop.prevent="saveRow(row.item)" currency="SEK" currency-symbol-position="suffix" thousand-separator=" "  v-model="row.item.edit_deposit_amount" class="form-control form-control-sm text-regular amount" :minus="true"></vue-numeric>
                    <span v-show="!row.item.is_edit"  @dblclick="editRow(row.item)">{{row.item.deposit_amount? row.item.deposit_amount.toLocaleString('sv-SE') : 0}} SEK </span>
                  </template>
                  <template slot="withdrawal_amount" slot-scope="row">
                    <vue-numeric v-show="row.item.is_edit"  @keyup.native.enter.stop.prevent="saveRow(row.item)" currency="SEK" currency-symbol-position="suffix" thousand-separator=" "  v-model="row.item.edit_withdrawal_amount" class="form-control form-control-sm text-regular amount" :minus="true"></vue-numeric>
                    <span v-show="!row.item.is_edit"  @dblclick="editRow(row.item)">{{row.item.withdrawal_amount ? row.item.withdrawal_amount.toLocaleString('sv-SE') : 0}} SEK </span>
                  </template>
                  <template slot="growth_amount" slot-scope="row">
                    <vue-numeric v-show="row.item.is_edit"  @keyup.native.enter.stop.prevent="saveRow(row.item)" currency="SEK" currency-symbol-position="suffix" thousand-separator=" "  v-model="row.item.edit_growth_amount" class="form-control form-control-sm text-regular amount" :minus="true"></vue-numeric>
                    <span v-show="!row.item.is_edit"  @dblclick="editRow(row.item)">{{row.item.growth_amount ? row.item.growth_amount.toLocaleString('sv-SE') : 0}} SEK </span>
                  </template>
                  <template slot="actions" slot-scope="row">
                    <div class="d-flex flex-column flex-md-row align-items-start">
                      <button v-show="row.item.is_edit" class='btn plain-btn text-regular' @click.stop="saveRow(row.item)">
                        <i class="flaticon stroke checkmark text-success"></i> Save
                      </button>
                      <button v-show="row.item.is_edit" class='btn plain-btn text-regular' @click.stop="cancelRow(row.item)">
                        <i class="fa fa-times text-warning"></i> Cancel
                      </button>
                      <button v-show="!row.item.is_edit" class='btn plain-btn text-regular' :disabled="customDisabled" @click.stop="editRow(row.item)">
                        <i class="flaticon solid edit-3 text-primary"></i> Edit
                      </button>
                      <button class='btn plain-btn text-regular' :disabled="customDisabled" @click.stop="deleteRow(row.item)">
                        <i class="flaticon stroke trash-2 text-danger"></i> Delete
                      </button>
                    </div>
                  </template>
                  <template slot="HEAD_actions" slot-scope="row">
                    <button class='btn btn-sm icon-btn text-regular add-row-btn' :disabled="customDisabled"  style="border-color: #eaecef;" @click.stop="openAddRowModal">
                      <i class="flaticon stroke plus text-primary"></i>
                      Add Row
                    </button>
                  </template>
                </b-table>
                <pagination :totalRows="totalRows" :currentPage.sync="currentPage" :perPage.sync="perPage"></pagination>
              </div>
            </b-tab>
            <template slot="tabs">
              <li class="nav-item select-tab-item" @click="tabIndex = tabIndex ? 0 : 1">
                <a class="flaticon solid up h-3"></a>
                <a class="flaticon solid down h-3"></a>
              </li>
            </template>
          </b-tabs>
        </b-card>
        <!-- asset Information Show Card -->
        <b-card class="depth-1 edit-info-card">
          <div class="car-icon">
            <i class="flaticon solid car-1"></i>
          </div>
          <b-container fluid>
            <b-row class="text-left">
              <b-col md="9">
                <b-row>
                  <b-col cols="6" md="4" xl="2" class="d-flex flex-column justify-content-center py-xl-0 py-2">
                    <span class="text-gray">Name</span>
                    <span class="text-regular font-weight-medium">{{asset.name}}</span>
                  </b-col>

                  <b-col cols="6" md="4" xl="2" class="d-flex flex-column justify-content-center py-xl-0 py-2">
                    <span class="text-gray">Category</span>
                    <span class="text-regular font-weight-medium" v-if="asset.category_id">{{categoryName}}</span>
                  </b-col>

                  <b-col cols="6" md="4" xl="2" class="d-flex flex-column justify-content-center py-xl-0 py-2">
                    <span class="text-gray">Type / Subtype</span>
                    <span class="text-regular font-weight-medium" v-if="asset.asset_subtype_id">{{subtypeName}}</span>
                  </b-col>

                  <b-col cols="6" md="4" xl="2" class="d-flex flex-column justify-content-center py-xl-0 py-2">
                    <span class="text-gray">Person</span>
                    <span class="text-regular font-weight-medium">{{personName}}</span>
                  </b-col>
                </b-row>
              </b-col>
              <b-col md="3" class="d-flex justify-content-center  justify-content-md-end align-items-center pt-5 py-md-0">
                  <button class='btn btn-sm icon-btn text-regular' style="border-color: #eaecef;" @click="openEditInfoModal">
                    <i class="flaticon solid edit-3 text-primary"></i>
                    Edit Info
                  </button>
              </b-col>
            </b-row>
          </b-container>
        </b-card>
      </div>
    </div>
    <!-- Add Row Modal -->
    <b-modal id="add-row-modal" class="livsplan-modal" size="lg" v-model="addRowModalShow" hide-footer>
      <div slot="modal-header" class="w-100 mx-auto">
        <h1> Add Row </h1>
        <button type="button" class="close" aria-label="Close" @click="addRowModalShow = false">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <b-form id="addRowForm" ref="addRowForm" novalidate :validated="addRowValidated" @submit.prevent="onAddRowSubmit">
        <b-container style="width: 60%">
          <b-row>
            <b-col sm="3" class="d-flex justify-content-end mt-3"><label :for="'year-input'">Year</label></b-col>
            <b-col sm="9">
              <b-form-group label-for="year-input" :state="yearState" :invalid-feedback="invalidYearFeedback" class="text-left">
                <b-form-input id="year-input" type="number" v-model="newRow.year" required :min="planStartYear" :max="planEndYear"></b-form-input>
              </b-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex justify-content-end"><label :for="'deposit-amount-input'">Deposit Amount</label></b-col>
            <b-col sm="9">
              <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="newRow.deposit_amount" class="form-control text-regular mb-3" :minus="true"></vue-numeric>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex justify-content-end"><label :for="'withdrawal-amount-input'">Withdrawal Amount</label></b-col>
            <b-col sm="9">
              <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="newRow.withdrawal_amount" class="form-control text-regular mb-3" :minus="true"></vue-numeric>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex justify-content-end"><label :for="'growth-amount-input'">Growth Amount</label></b-col>
            <b-col sm="9">
              <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="newRow.growth_amount" class="form-control text-regular mb-3" :minus="true"></vue-numeric>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex justify-content-end mt-3"><label :for="'amount-input'">Amount</label></b-col>
            <b-col sm="9">
              <vue-numeric currency="SEK" currency-symbol-position="suffix" thousand-separator=" " v-model="newRow.amount" class="form-control text-regular mb-3" :minus="true"></vue-numeric>
            </b-col>
          </b-row>
          <div class="w-100 mx-auto">
            <b-btn type="submit" size="lg" variant="primary">
              Save
            </b-btn>
          </div>
        </b-container>
      </b-form>
    </b-modal>
    <!-- Edit Info Modal -->
    <b-modal id="edit-info-modal" class="livsplan-modal" size="lg" v-model="editInfoModalShow" hide-footer>
      <div slot="modal-header" class="w-100 mx-auto">
        <h1>Edit: {{asset.name}}</h1>
        <button type="button" class="close" aria-label="Close" @click="closeEditInfoModal">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <b-form id="editInfoForm" ref="editInfoForm" novalidate :validated="editInfoValidated" @submit.prevent="onEditInfoSumbit">
        <b-container style="width: 60%">
          <b-row>
            <b-col sm="3" class="d-flex justify-content-end mt-3"><label :for="'nameInput'">Name</label></b-col>
            <b-col sm="9">
              <b-form-group label-for="nameInput" :state="nameState" class="text-left">
                <b-form-input id="nameInput"  v-model="editAsset.name" :state="nameState" aria-describedby="nameInputFeedback" required></b-form-input>
                <b-form-invalid-feedback id="nameInputFeedback">
                    Name is required and minium length is 2
                </b-form-invalid-feedback>
              </b-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex mt-3 justify-content-end"><label :for="'subtype-select'">Subtype</label></b-col>
            <b-col sm="9">
              <b-form-group label-for="subtype-select" class="text-left">
                <b-form-select :options="subtypeOptions" :value-field="'subtype_id'" :text-field="'subtype'" v-model="editAsset.asset_subtype_id" :id="'subtype-select'" />
              </b-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex mt-3 justify-content-end"><label :for="'category-select'">Category</label></b-col>
            <b-col sm="9">
              <b-form-group label-for="category-select" :state="categoryState" class="text-left">
                <b-form-select :options="categoryOptions" :value-field="'id'" :text-field="'name'" v-model="editAsset.category_id" :id="'category-select'" :state="categoryState" aria-describedby="categorySelectFeedback" required/>
                <b-form-invalid-feedback id="categorySelectFeedback">
                    This field is required
                </b-form-invalid-feedback>
              </b-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex mt-3 justify-content-end"><label :for="'person-select'">Person</label></b-col>
            <b-col sm="9">
              <b-form-group label-for="person-select" :state="personState" class="text-left">
                <b-form-select :options="personOptions" :value-field="'id'" :text-field="'name'" v-model="editAsset.person_id" :id="'person-select'" :state="personState" aria-describedby="personSelectFeedback" required/>
                <b-form-invalid-feedback id="personSelectFeedback">
                    This field is required
                </b-form-invalid-feedback>
              </b-form-group>
            </b-col>
          </b-row>
          <b-row>
            <b-col sm="3" class="d-flex mt-3 justify-content-end"><label :for="'currency-select'">Currency</label></b-col>
            <b-col sm="9">
              <b-form-group label-for="currency-select" class="text-left">
                <b-form-select :options="currencyOptions" :value-field="'id'" :text-field="'name'" v-model="editAsset.currency_id" :id="'currency-select'" />
              </b-form-group>
            </b-col>
          </b-row>
          <div class="w-100 mx-auto">
            <b-btn type="submit" size="lg" variant="primary">
              Save
            </b-btn>
          </div>
        </b-container>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
import axios from 'axios'
import Switches from 'vue-switches'
import barchart from '@/components/charts/barchart'
import EventBus from '@/event-bus.js'
import pagination from '@/components/common/pagination'

export default {
  name: 'viewasset',
  async mounted () {
    try {
      let response = await axios.get(`${process.env.ROOT_API}/assets/${this.$route.params.id}`)
      this.asset = response.data
      this.plan = this.asset
      response = await axios.get(`${process.env.ROOT_API}/plans/${this.asset.plan_id}`)
      this.planStartYear = response.data.start_year
      this.planEndYear = response.data.end_year
      if (this.asset.calculation_mode === 'auto') {
        this.customDisabled = true
        this.tabIndex = 0
      } else {
        this.customDisabled = false
        this.tabIndex = 1
      }
      response = await axios.get(`${process.env.ROOT_API}/categories`)
      this.categoryOptions = response.data
      response = await axios.get(`${process.env.ROOT_API}/persons`)
      this.personOptions = response.data
      response = await axios.get(`${process.env.ROOT_API}/assets/subtypes`)
      this.subtypeOptions = response.data
      response = await axios.get(`${process.env.ROOT_API}/currencies`)
      this.currencyOptions = response.data
      EventBus.$emit('select-plan', {
        plan_id: this.asset.plan_id
      })
    } catch (err) {
      console.log(err)
    }
  },
  data () {
    return {
      asset: {},
      editAsset: {},
      newRow: {},
      categoryOptions: [],
      personOptions: [],
      currencyOptions: [],
      subtypeOptions: [],
      fields: [
        { key: 'year', label: 'Year', sortable: true, sortDirection: 'desc' },
        { key: 'amount', label: 'Amount', sortable: true },
        { key: 'deposit_amount', label: 'Deposit Amount', sortable: true },
        { key: 'withdrawal_amount', label: 'Withdrawal Amount', sortable: true },
        { key: 'growth_amount', label: 'Growth Amount', sortable: true },
        { key: 'actions', 'class': 'd-lg-flex justify-content-lg-end align-items-center' }
      ],
      currentPage: 1,
      perPage: this.$store.getters.perPage ? this.$store.getters.perPage : 10,
      filter: null,
      sortBy: null,
      sortDesc: false,
      sortDirection: 'asc',
      totalRows: 0,
      editInfoModalShow: false,
      addRowModalShow: false,
      isCalcSaveDisabled: true,
      autoCalcValidated: false,
      editInfoValidated: false,
      addRowValidated: false,
      isSaving: false,
      customDisabled: true,
      tabIndex: 0,
      plan: {},
      planStartYear: null,
      planEndYear: null
    }
  },
  computed: {
    items () {
      return [{
        text: 'Assumptions',
        to: { name: 'assumptions' }
      }, {
        text: 'assets',
        to: { name: 'assets' }
      }, {
        text: this.asset.name,
        active: true
      }]
    },
    startYearState () {
      return !this.autoCalcValidated || +this.asset.start_year >= +this.planStartYear && +this.asset.start_year < +this.asset.end_year
    },
    invalidStartYearFeedback () {
      if (+this.asset.start_year >= +this.planStartYear) {
        if (+this.asset.start_year >= +this.asset.end_year) {
          return 'Start year must be less than end year'
        } else {
          return ''
        }
      } else {
        return `Star year is required and must be at least ${this.planStartYear}`
      }
    },
    endYearState () {
      return !this.autoCalcValidated || +this.asset.end_year > +this.planStartYear && +this.asset.end_year <= +this.planEndYear
    },
    invalidEndYearFeedback () {
      if (+this.asset.end_year <= +this.planEndYear) {
        if (+this.asset.end_year <= +this.planStartYear) {
          return `End year must be larger than ${this.planStartYear}`
        } else {
          return ''
        }
      } else {
        return `End year must be less than ${this.planEndYear + 1}`
      }
    },
    yearState () {
      return !this.addRowValidated || (+this.newRow.year >= +this.planStartYear && +this.newRow.year <= +this.planEndYear)
    },
    invalidYearFeedback () {
      if (+this.newRow.year >= +this.planStartYear && +this.newRow.year <= +this.planEndYear) {
        return ''
      } else {
        return `Year must be between ${this.planStartYear} and ${this.planEndYear}`
      }
    },
    nameState () {
      if (!this.editInfoValidated) {
        return null
      } else {
        return this.editAsset.name && this.editAsset.name.length > 1
      }
    },
    categoryState () {
      if (!this.editInfoValidated) {
        return null
      } else {
        return this.editAsset.category_id != null
      }
    },
    personState () {
      if (!this.editInfoValidated) {
        return null
      } else {
        return this.editAsset.person_id != null
      }
    },
    personName () {
      let f = this.personOptions.filter(option => option.id === this.asset.person_id)
      if (f.length) {
        return f[0].name
      }
    },
    categoryName () {
      let f = this.categoryOptions.filter(option => option.id === this.asset.category_id)
      if (f.length) {
        return f[0].name
      }
    },
    subtypeName () {
      let f = this.subtypeOptions.filter(option => option.subtype_id === this.asset.asset_subtype_id)
      if (f.length) {
        return f[0].subtype
      }
    }
  },
  methods: {
    onFiltered (filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
    autoCalcformChanged  (ev) {
      this.isCalcSaveDisabled = false
    },
    async onAutoCalcSumbit (ev) {
      this.autoCalcValidated = true
      if (this.$refs.autoCalcForm.checkValidity() === true) {
        this.isCalcSaveDisabled = true
        this.autoCalcValidated = false
        this.isSaving = true
        this.$root.$emit('bv::hide::tooltip')
        try {
          let data = {
            start_year: this.asset.start_year,
            end_year: this.asset.end_year,
            amount: this.asset.amount,
            deposit_amount: this.asset.deposit_amount,
            withdrawal_amount: this.asset.withdrawal_amount,
            annual_growth_rate: this.asset.annual_growth_rate
          }
          let response = await axios.put(`${process.env.ROOT_API}/assets/${this.$route.params.id}`, data)
          this.asset.asset_amounts = this.convertToArray(response.data.asset_amounts)
          this.isSaving = false
          EventBus.$emit('notify-me', {
            title: 'Success!',
            text: 'New vaules have been saved successfully!',
            status: 'is-success'
          })
        } catch (err) {
          console.log(err)
          this.isSaving = false
          EventBus.$emit('notify-me', {
            title: 'Failed!',
            text: err.message,
            status: 'is-danger'
          })
        }
      }
    },
    openEditInfoModal () {
      this.autoCalcValidated = false
      this.editInfoValidated = false
      this.editInfoModalShow = true
      this.editAsset = {...this.asset}
    },
    closeEditInfoModal () {
      this.editInfoModalShow = false
      this.editAsset = {}
    },
    async onEditInfoSumbit (ev) {
      this.editInfoValidated = true
      if (this.$refs.editInfoForm.checkValidity() === true) {
        try {
          let data = {
            name: this.editAsset.name,
            category_id: this.editAsset.category_id,
            person_id: this.editAsset.person_id,
            asset_subtype_id: this.editAsset.asset_subtype_id,
            currency_id: this.editAsset.currency_id
          }
          await axios.put(`${process.env.ROOT_API}/assets/${this.$route.params.id}`, data)
          EventBus.$emit('notify-me', {
            title: 'Success!',
            text: 'New vaules have been saved successfully!',
            status: 'is-success'
          })
          this.editInfoModalShow = false
          let response = await axios.get(`${process.env.ROOT_API}/assets/${this.$route.params.id}`)
          this.asset = response.data
        } catch (err) {
          console.log(err)
          EventBus.$emit('notify-me', {
            title: 'Failed!',
            text: err.message,
            status: 'is-danger'
          })
        }
      }
    },
    openAddRowModal () {
      this.addRowValidated = false
      this.addRowModalShow = true
      this.newRow = {}
    },
    async onAddRowSubmit (ev) {
      this.addRowValidated = true
      if (this.$refs.addRowForm.checkValidity() === true) {
        try {
          let data = {
            asset_id: this.asset.id,
            year: this.newRow.year,
            amount: this.newRow.amount ? this.newRow.amount : 0,
            deposit_amount: this.newRow.deposit_amount ? this.newRow.deposit_amount : 0,
            withdrawal_amount: this.newRow.withdrawal_amount ? this.newRow.withdrawal_amount : 0,
            growth_amount: this.newRow.growth_amount ? this.newRow.growth_amount : 0
          }
          let response = await axios.post(`${process.env.ROOT_API}/assets/${this.$route.params.id}/amounts`, data)
          let found = this.asset.asset_amounts.find(element => parseInt(element.year) === parseInt(response.data.year))
          if (found) {
            found.amount = response.data.amount
            found.deposit_amount = response.data.deposit_amount
            found.withdrawal_amount = response.data.withdrawal_amount
            found.growth_amount = response.data.growth_amount
          } else {
            this.asset.asset_amounts.unshift(response.data)
          }
          this.addRowModalShow = false
          this.plan = this.asset
          EventBus.$emit('notify-me', {
            title: 'Success!',
            text: 'New Row has been added successfully!',
            status: 'is-success'
          })
        } catch (err) {
          console.log(err)
          EventBus.$emit('notify-me', {
            title: 'Failed!',
            text: err.message,
            status: 'is-danger'
          })
        }
      }
    },
    editRow (item) {
      if (!this.customDisabled) {
        this.$set(item, 'is_edit', true)
        item.edit_amount = item.amount
        item.edit_deposit_amount = item.deposit_amount
        item.edit_withdrawal_amount = item.withdrawal_amount
        item.edit_growth_amount = item.growth_amount
      }
    },
    cancelRow (item) {
      item.is_edit = false
    },
    async saveRow (item) {
      let data = {
        amount: item.edit_amount,
        deposit_amount: item.edit_deposit_amount,
        withdrawal_amount: item.edit_withdrawal_amount,
        growth_amount: item.edit_growth_amount
      }
      await axios.put(`${process.env.ROOT_API}/assets/${this.$route.params.id}/amounts/${item.id}`, data)
      item.amount = item.edit_amount
      item.deposit_amount = item.edit_deposit_amount
      item.withdrawal_amount = item.edit_withdrawal_amount
      item.growth_amount = item.edit_growth_amount
      item.is_edit = false
      this.plan = this.asset
    },
    deleteRow (item) {
      let message = {
        title: 'Confirm',
        body: 'Are you sure you want to delete this row?'
      }
      this.$dialog.confirm(message)
      .then(async () => {
        await axios.delete(`${process.env.ROOT_API}/assets/${this.$route.params.id}/amounts/${item.id}`)
        this.asset.asset_amounts = this.asset.asset_amounts.filter(amount => amount.id !== item.id)
        this.plan = this.asset
      })
    },
    deleteAsset () {
      let message = {
        title: 'Confirm',
        body: 'Are you sure you want to delete this asset?'
      }
      this.$dialog.confirm(message)
      .then(async () => {
        await axios.delete(`${process.env.ROOT_API}/assets/${this.$route.params.id}`)
        this.$router.push('/assumptions/assets')
        EventBus.$emit('notify-me', {
          title: 'Success!',
          text: 'This asset has been deleted successfully!',
          status: 'is-success'
        })
      })
    },
    convertToArray (obj) {
      let arr = []
      for (let prop in obj) {
        arr.push(obj[prop])
      }
      return arr
    },
    async recalculateChart () {
      if (this.asset.calculation_mode === 'auto') {
        if (this.asset.start_year && +this.asset.start_year >= +this.planStartYear) {
          if (this.asset.end_year && +this.asset.end_year <= +this.planEndYear) {
            if (+this.asset.end_year > +this.asset.start_year) {
              if (this.asset.amount != null && this.asset.deposit_amount != null && this.asset.withdrawal_amount != null && this.asset.annual_growth_rate != null) {
                let data = {
                  start_year: this.asset.start_year,
                  end_year: this.asset.end_year,
                  amount: +this.asset.amount,
                  deposit_amount: +this.asset.deposit_amount,
                  withdrawal_amount: +this.asset.withdrawal_amount,
                  annual_growth_rate: +this.asset.annual_growth_rate,
                  calculation_mode: 'auto'
                }
                let response = await axios.post(`${process.env.ROOT_API}/assets/amounts/calculate`, data)
                response.data.asset_amounts = this.convertToArray(response.data.asset_amounts)
                this.plan = response.data
              }
            }
          }
        }
      }
    }
  },
  watch: {
    customDisabled: async function (val) {
      let modeChanged = false
      if (val && this.asset.calculation_mode === 'custom') {
        modeChanged = true
      } else if (!val && this.asset.calculation_mode === 'auto') {
        modeChanged = true
      }

      if (modeChanged) {
        let data = {
          calculation_mode: val ? 'auto' : 'custom'
        }
        try {
          await axios.put(`${process.env.ROOT_API}/assets/${this.$route.params.id}`, data)
          this.asset.calculation_mode = val ? 'auto' : 'custom'
          EventBus.$emit('notify-me', {
            title: 'Success!',
            text: 'Calculation mode has been updated successfully!',
            status: 'is-success'
          })
        } catch (err) {
          console.log(err)
          EventBus.$emit('notify-me', {
            title: 'Failed!',
            text: err.message,
            status: 'is-danger'
          })
        }
      }
    },
    'asset.start_year': function (val) {
      this.recalculateChart(this)
    },
    'asset.end_year': function (val) {
      this.recalculateChart(this)
    },
    'asset.amount': function (val) {
      this.recalculateChart(this)
    },
    'asset.deposit_amount': function (val) {
      this.recalculateChart(this)
    },
    'asset.withdrawal_amount': function (val) {
      this.recalculateChart(this)
    },
    'asset.annual_growth_rate': function (val) {
      this.recalculateChart(this)
    }
  },
  components: {
    barchart,
    Switches,
    pagination
  }
}
</script>

<style lang="scss">
  @import './style.scss'
</style>
