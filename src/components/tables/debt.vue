<template>
  <div class="debt-table">
    <b-table show-empty
            stacked="md"
            :items="debts"
            :fields="fields"
            :current-page="currentPage"
            :per-page="perPage"
            :sort-by.sync="sortBy"
            :sort-desc.sync="sortDesc"
            :sort-direction="sortDirection"
            :filter="filter"
            @filtered="onFiltered"
            class="depth-1"
    >
    <template slot="top-row" slot-scope="data">
      <td colspan="7">
        <div class="d-flex align-items-center">
          <i class="fa fa-search"></i>
          <b-form-input v-model="filter" size="sm" placeholder="Type to Search" />
        </div>
      </td>
    </template>
    <template slot="name" slot-scope="row">
      <div class="d-flex">
        <i class="flaticon solid star-2"></i>
        <b-link class="font-weight-bold text-regular" @click.stop="gotoDetail(row.item.id)">{{row.item.name}}</b-link>
      </div>
    </template>
    <template slot="amount" slot-scope="row">
      <span>{{row.item.amount.toLocaleString('sv-SE')}} SEK </span>
    </template>
    <template slot="period" slot-scope="row">
      <span>{{row.item.start_year}} - {{row.item.end_year}}</span>
    </template>
      <template slot="amortization_amount" slot-scope="row">
      <span v-if="row.item.amortization_amount">{{row.item.amortization_amount.toLocaleString('sv-SE')}} SEK </span>
    </template>
    <template slot="actions" slot-scope="row">
      <button class='btn plain-btn text-regular mr-3' @click.stop="gotoDetail(row.item.id)">
        <i class="flaticon solid right-circle-2 text-primary"></i> Open
      </button>
    </template>
    <template slot="HEAD_name" slot-scope="row">
      <div class="d-flex align-items-center">
        <div class="table-title-label"></div>
        <i class="flaticon stroke document text-danger table-title-label-icon"></i>
        <span class="table-title">Debts</span>
      </div>
    </template>
    <template slot="HEAD_actions" slot-scope="row">
      <button class='btn btn-sm btn-primary font-weight-bold add-new-row d-none d-sm-block' @click.stop="$router.push('/assumptions/debts/add-debt')">
        <i class="flaticon stroke plus"></i>
        Add New Debt
      </button>
      <button class='btn btn-sm btn-primary font-weight-bold add-new-row d-block d-sm-none'>
        <i class="flaticon stroke plus"></i>
      </button>
    </template>
    </b-table>
    <pagination :totalRows="totalRows" :currentPage.sync="currentPage" :perPage.sync="perPage"></pagination>
  </div>
</template>

<script>
import axios from 'axios'
import pagination from '@/components/common/pagination'

export default {
  name: 'debtTable',
  data () {
    return {
      debts: [],
      fields: [
        { key: 'name', label: 'Debts', sortable: true },
        { key: 'period', label: 'Period', sortable: true },
        { key: 'amount', label: 'Amount', sortable: true },
        { key: 'interest_rate', label: 'Interest Rate', sortable: true },
        { key: 'amortization_amount', label: 'Amortization Amount', sortable: true },
        { key: 'person_id',
          label: 'Person',
          formatter: (value) => {
            let f = this.personOptions.filter(option => option.id === value)
            if (f.length) {
              return f[0].name
            }
          },
          sortable: true
        },
        { key: 'actions', 'class': 'd-lg-flex justify-content-lg-end align-items-center' }
      ],
      currentPage: 1,
      perPage: this.$store.getters.perPage ? this.$store.getters.perPage : 10,
      sortBy: null,
      sortDesc: false,
      sortDirection: 'asc',
      pageOptions: [ 5, 10, 15, 25 ],
      totalRows: 0,
      filter: null,
      personOptions: []
    }
  },
  async mounted () {
    try {
      let response = await axios.get(`${process.env.ROOT_API}/debts`)
      this.debts = response.data.data
      this.totalRows = this.debts.length
      response = await axios.get(`${process.env.ROOT_API}/persons`)
      this.personOptions = response.data
    } catch (error) {
      console.error(error)
    }
  },
  methods: {
    gotoDetail (id) {
      this.$router.push(`/assumptions/debts/${id}`)
    },
    onFiltered (filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    }
  },
  components: {
    pagination
  }
}
</script>
<style lang="scss">
  .debt-table {
      table {
        background: white;

        @media (max-width: 767.99px) {
          thead {
            display: block !important;

            tr {
              display: flex;
              justify-content: space-between;
            }
          }

          .add-new-row {
            margin: 5px 8px !important;
          }

          .table-title-label {
            margin-left: 2px !important;
            margin-right: 10px !important;
          }

        }

        @media (max-width: 575.98px) {
          .add-new-row {
            margin: 10px 8px !important;
            padding-left: 7px !important;
            padding-right: 7px !important;
          }

          .table-title {
            margin-top: 8px;
          }

          .table-title-label {
            margin-top: 16px !important;
          }
        }


        thead {
          th {
            border: none;
            vertical-align: middle;
            outline: none;

            .table-title {
              font-size: 24px;
              line-height: 32px;
              color: #434343;
            }

            .table-title-label {
              border-radius: 0 !important;
              width: 21px !important;
              height: 24px;
              border: 3px solid #D8D8D8;
              margin-left: 12px;
              margin-right: 12px;
              margin-top: 10px;
            }

            .table-title-label-icon {
              position: absolute;
              left: 30px;
              font-size: 21px;
            }

            .add-new-row {
              border-radius: 30px;
              margin: 30px;
              padding-left: 12px;
              padding-right: 16px;
            }

            &.sorting::before {
              top: 38px !important;
              margin-left: 0.45em !important;
              vertical-align: 0.1em !important;
              border-right: 0.5em solid transparent !important;
              border-left: 0.5em solid transparent !important;
              border-top: 0 !important;
              border-bottom: .5em solid !important;
              content: "" !important;
              width: 0 !important;
              height: 0 !important;
              border-radius: 15% !important;
            }

            &.sorting::after {
              top: 62px !important;
              font-size: 24px !important;
              margin-left: 0.255em !important;
              vertical-align: 0.255em !important;
              border-top: 0.3em solid !important;
              border-right: 0.3em solid transparent !important;
              border-bottom: 0 !important;
              border-left: 0.3em solid transparent !important;
              content: "" !important;
              width: 0 !important;
              height: 0 !important;
              border-radius: 15% !important;
            }

            &.sorting_desc::before, &.sorting_asc::after {
              opacity: 1;
              color: #36b37E;
            }

            @media (max-width: 769px) {
              &.sorting::after, &.sorting::before {
                display: none !important;
              }
            }

            &[aria-colindex="1"] {
              &.sorting::before, &.sorting::after {
                left: 128px;
              }
            }
            &[aria-colindex="2"] {
              &.sorting::before, &.sorting::after {
                left: 56px;
              }
            }
            &[aria-colindex="3"] {
              &.sorting::before, &.sorting::after {
                left: 68px;
              }
            }
            &[aria-colindex="4"] {
              &.sorting::before, &.sorting::after {
                left: 102px;
              }
            }
            &[aria-colindex="5"] {
              &.sorting::before, &.sorting::after {
                left: 162px;
              }
            }
            &[aria-colindex="6"] {
              &.sorting::before, &.sorting::after {
                left: 62px;
              }
            }

          }
        }

        tbody {
          background: linear-gradient(0deg, #FFFFFF 0%, #F8F8F8 100%);

          :first-child {
            @media (min-width: 769px) {
              td {
                border: none !important;
              }
            }
          }

          tr {
            td {
              vertical-align: middle;
              border-top: 1px dashed #CACACA;
            }

            &.b-table-top-row {
              input {
                border: none;
              }

              i {
                font-size: 22px;
                color: #A5ADBA;
                margin-left: 12px;
                margin-right: 12px;
              }
            }
          }

          .star-2 {
            color: #c1c7D0;
            margin-right: 14px;
            @media (min-width: 769px) {
              margin-left: 14px;
            }
          }


        }

        @media (max-width: 767.99px) {
          &.b-table.b-table-stacked-md > tbody > tr > :first-child {
            border-top-style: solid;
            border-top-width: 0.3rem;
          }
        }

        @media (max-width: 767.99px) {
          thead {
            tr {
              > :nth-child(2) {
                display: none;
              }

              > :nth-child(3) {
                display: none;
              }

              > :nth-child(4) {
                display: none;
              }

              > :nth-child(5) {
                display: none;
              }

              > :nth-child(6) {
                display: none;
              }
            }
          }

          .table-title-label-icon {
            left: 20px !important
          }
        }

        @media (max-width: 575.98px) {

          .table-title-label-icon {
            top: 21px !important;
          }
        }
      }
  }
</style>

