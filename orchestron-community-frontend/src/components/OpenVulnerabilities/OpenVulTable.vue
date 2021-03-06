<template>
    <div>
        <b-container fluid style="background-color: #FFFFFF;">
            <b-col cols="12">
                <br>
                <b-row>
                    <b-col md="6" class="my-1">
                        <b-form-input v-model="filter" placeholder="Type to Search" class="inline-form-control"/>
                    </b-col>
                    <b-col md="2"></b-col>
                    <b-col md="4" class="my-1">
                    </b-col>
                </b-row>
                <b-row>
                    <b-col md="6" class="my-1">
                    </b-col>
                    <b-col md="6" class="my-1">
                        <b-pagination :total-rows="numPages"
                            :per-page="perPage"
                            v-model="currentPage" class="my-1" align="right" @input="paginationClick(currentPage)"></b-pagination>
                    </b-col>
                </b-row>
                <b-table show-empty
                    stacked="md"
                    :items="dataItems"
                    :fields="fields"
                    :current-page="currentPage"
                    :per-page="perPage"
                    :filter="filter"
                    :sort-by.sync="sortBy"
                    :sort-desc.sync="sortDesc"
                    class="m2_top">
                    <template slot="sev" slot-scope="row">
                        <div class="high-vul-severity-line" v-if="row.item.sev===3"></div>
                        <div class="medium-vul-severity-line" v-if="row.item.sev===2"></div>
                        <div class="low-vul-severity-line" v-if="row.item.sev===1"></div>
                        <div class="info-vul-severity-line" v-if="row.item.sev===0"></div>
                    </template>
                    <template slot="name" slot-scope="row">
                        <p  v-if="row.item.multiple" class="title" v-b-tooltip.hover :title="row.item.commonName">{{row.item.commonName}} - Multiple</p>
                        <p  v-else="!row.item.multiple" class="title" v-b-tooltip.hover :title="row.item.commonName">{{row.item.commonName}}</p>
                        <b-row>
                            <b-col cols="4">
                                <p>
                                    <span class="sub-title">CWE</span>
                                    <span class="sub-divider">:</span>
                                    <span class="sub-value">{{ row.item.cwe }}</span>
                                </p>
                            </b-col>
                            <b-col cols="4">
                                <p>
                                    <span class="sub-title">Open For</span>
                                    <span class="sub-divider">:</span>
                                    <span class="sub-value" >{{ row.item.openFor }} days ago</span>
                                </p>
                            </b-col>
                        </b-row>
                    </template>
                    <template slot="actions" slot-scope="row">
                        <p><span class="second-sub-title">Tools : </span> <span class="second-sub-name">{{ row.item.tool }}</span></p>
                        <p>
                            <span class="second-sub-title" v-if="!row.item.multiple">Apps : </span>
                            <b-button size="sm" @click="viewIndividualVul(row.item.app, row.item.vulName, row.item.cwe)" class="mr-1 btn-orange"
                                v-if="!row.item.multiple">
                                {{ row.item.app }}
                            </b-button>
                        </p>
                        <b-button size="sm" @click.stop="row.toggleDetails" class="btn-orange" v-if="row.item.multiple">
                            {{ row.detailsShowing ? 'Hide' : 'Show' }}-Details
                        </b-button>
                    </template>
                    <template slot="row-details" slot-scope="row" v-if="row.item.multiple">
                        <b-card >
                            <template v-for="(value, key) in row.item.name">
                                <b-row>
                                    <b-col>
                                        <b-list-group>
                                            <b-list-group-item class="d-flex justify-content-between align-items-center">
                                                {{ key }}
                                                <b-button size="sm" @click="viewIndividualVul(value, key, row.item.cwe)" class="mr-1 btn-orange">
                                                    {{ value }}
                                                </b-button>
                                            </b-list-group-item>
                                            <br>
                                        </b-list-group>
                                    </b-col>
                                </b-row>
                            </template>
                        </b-card>
                    </template>
                </b-table>
            </b-col>
        </b-container>
    </div>
</template>

<script>
const items = []

export default {
  name: 'OpenVulTable',
  data() {
    return {
      counter: 45,
      max: 200,
      items: '',
      fields: [
        { key: 'sev', label: ' ', sortable: true, 'class': 'title' },
        { key: 'name', label: 'Vulnerability Name', sortable: true, 'class': 'title' },
        { key: 'actions', label: ' ', 'class': 'title', sortable: false }
      ],
      currentPage: 0,
      perPage: 5,
      totalRows: 0,
      pageOptions: [5, 10, 15],
      sortBy: null,
      sortDesc: false,
      filter: null,
      numPages: 0
    }
  },
  props: {
    dataItems: {
      type: Array,
      required: true
    },
    pageNumberCount: {
      required: false
    },
    pageCurrent: {
      required: false
    }
  },
  beforeUpdate() {
    this.items = this.dataItems
    this.numPages = this.pageNumberCount
    this.currentPage = this.pageCurrent
  },
  computed: {
    sortOptions() {
      return this.fields
        .filter(f => f.sortable)
        .map(f => { return { text: f.label, value: f.key } })
    }
  },
  methods: {
    viewIndividualVul(app, name, cwe) {
      const encodedApp = btoa(unescape(encodeURIComponent(app)))
      const encodedName = btoa(unescape(encodeURIComponent(name)))
      const encodedCwe = btoa(unescape(encodeURIComponent(cwe)))
      this.$router.push({ path: '/org/individual_vul/' + encodedApp + '/' + encodedName + '/' + encodedCwe })
    },
    paginationClick(currentPage) {
      this.$emit('clickPagination', { page: currentPage })
    }
  }
}
</script>

<style scoped>
  .title {
    font-family: 'Avenir';
    font-size: 14px;
    white-space: nowrap;
    overflow: hidden;
    width: 500px;
    text-overflow: ellipsis;
    text-align: left;
  }

  .high-vul-severity-line {
    width: 5px;
    height: 38px;
    background-color: #d11d55;
    margin-top: 15%;
  }

  .medium-vul-severity-line {
    width: 5px;
    height: 38px;
    background-color: #ff9c2c;
    margin-top: 15%;
  }

  .low-vul-severity-line {
    width: 5px;
    height: 38px;
    background-color: #008b8f;
    margin-top: 15%;
  }

  .info-vul-severity-line {
    width: 5px;
    height: 38px;
    background-color: #1d1e52;
    margin-top: 15%;
  }

  .btn-orange {
    color: #ff542c;
    background-color: #FFFFFF;
    border-color: #ff542c;
    font-family: 'Avenir';
    border-radius: 14px;
    padding: 3px 12px;
    margin-bottom: 0;
    font-size: 14px;
  }

  .btn-orange:focus,
  .btn-orange.focus {
    color: #ff542c;
    background-color: #FFFFFF;
    border-color: #ff542c;
    font-family: 'Avenir';
    border-radius: 14px;
    padding: 3px 12px;
    margin-bottom: 0;
    font-size: 14px;
  }

  .btn-orange:hover {
    color: #FFFFFF;
    background-color: #ff542c;
    border-color: #FFFFFF;
    font-family: 'Avenir';
    border-radius: 14px;
    padding: 3px 12px;
    margin-bottom: 0;
    font-size: 14px;
  }

  .sub-title {
    width: 200px;
    height: 12px;
    font-family: 'Avenir';
    font-size: 12px;
    font-weight: 600;
    line-height: 1.33;
    text-align: left;
    color: #6b7784;
  }

  .sub-divider {
    width: 10px;
    height: 12px;
    font-family: 'Avenir';
    font-size: 12px;
    font-weight: 600;
    line-height: 1.33;
    text-align: center;
    color: #6b7784;
  }

  .sub-value {
    width: 100px;
    height: 12px;
    font-family: 'Avenir';
    font-size: 12px;
    font-weight: 600;
    line-height: 1.33;
    text-align: center;
    color: #6b7784;
  }

  .second-sub-title {
    font-family: 'Avenir';
    font-size: 12px;
    font-weight: 600;
    line-height: 1.5;
    color: #979797;
  }

  .second-sub-name {
    font-family: 'Avenir';
    font-size: 12px;
  }

  .inline-form-control {
    box-sizing: border-box;
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    outline: none;
    width: 100%;
    padding: 7px;
    border: none;
    border-bottom: 1px solid #ddd;
    background: transparent;
    margin-bottom: 10px;
    font: 16px 'Avenir';
    height: 45px;
    position: relative;
    display: inline-block;
  }
</style>
