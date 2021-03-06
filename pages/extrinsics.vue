<template>
  <div>
    <section>
      <b-container class="main py-5">
        <h1 class="mb-4">
          {{ $t("pages.extrinsics.title") }}
        </h1>
        <div class="last-extrinsics">
          <!-- Filter -->
          <b-row style="margin-bottom: 1rem">
            <b-col cols="12">
              <b-form-input
                id="filterInput"
                v-model="filter"
                type="search"
                :placeholder="$t('pages.blocks.search_placeholder')"
              />
            </b-col>
          </b-row>
          <div class="table-responsive">
            <b-table striped hover :fields="fields" :items="extrinsics">
              <template v-slot:cell(block_number)="data">
                <p class="mb-0">
                  <nuxt-link
                    v-b-tooltip.hover
                    :to="
                      `/extrinsic/${data.item.block_number}/${data.item.extrinsic_index}`
                    "
                    title="Check extrinsic information"
                  >
                    {{ data.item.block_number }}-{{ data.item.extrinsic_index }}
                  </nuxt-link>
                </p>
              </template>
              <template v-slot:cell(hash)="data">
                <p class="mb-0">{{ shortHash(data.item.hash) }}</p>
              </template>
              <template v-slot:cell(section)="data">
                <p class="mb-0">
                  {{ data.item.section }} ➡
                  {{ data.item.method }}
                </p>
              </template>
              <template v-slot:cell(success)="data">
                <p class="mb-0">
                  <i
                    v-if="data.item.success"
                    class="fa fa-check-circle text-success"
                    aria-hidden="true"
                  ></i>
                  <i
                    v-else
                    class="fa fa-check-circle text-danger"
                    aria-hidden="true"
                  ></i>
                </p>
              </template>
            </b-table>
            <div class="mt-2" style="display: flex">
              <b-pagination
                v-model="page"
                :total-rows="totalRows"
                :per-page="perPage"
              />
              <b-button-group class="mx-4">
                <b-button
                  v-for="(item, index) in tableOptions"
                  :key="index"
                  @click="handleNumFields(item)"
                >
                  {{ item }}
                </b-button>
              </b-button-group>
            </div>
          </div>
        </div>
      </b-container>
    </section>
  </div>
</template>

<script>
import commonMixin from "../mixins/commonMixin.js";
import Identicon from "../components/identicon.vue";
import gql from "graphql-tag";
import { network, paginationOptions } from "../polkastats.config.js";

export default {
  mixins: [commonMixin],
  data: function() {
    return {
      filter: "",
      extrinsics: [],
      tableOptions: paginationOptions,
      perPage: localStorage.paginationOptions
        ? parseInt(localStorage.paginationOptions)
        : 10,
      page: 1,
      totalRows: 1,
      fields: [
        {
          key: "block_number",
          label: "Extrinsic",
          sortable: true
        },
        {
          key: "hash",
          label: "Hash",
          sortable: true
        },
        {
          key: "section",
          label: "Extrinsic",
          sortable: true
        },
        {
          key: "success",
          label: "Success",
          sortable: true
        }
      ]
    };
  },
  methods: {
    handleNumFields(num) {
      localStorage.paginationOptions = num;
      this.perPage = parseInt(num);
    }
  },
  apollo: {
    $subscribe: {
      extrinsic: {
        query: gql`
          subscription extrinsics(
            $blockNumber: bigint
            $perPage: Int!
            $offset: Int!
          ) {
            extrinsic(
              limit: $perPage
              offset: $offset
              where: { block_number: { _eq: $blockNumber } }
              order_by: { block_number: desc, extrinsic_index: desc }
            ) {
              block_number
              extrinsic_index
              is_signed
              signer
              section
              method
              hash
              success
            }
          }
        `,
        variables() {
          return {
            blockNumber: this.filter ? parseInt(this.filter) : undefined,
            perPage: this.perPage,
            offset: (this.page - 1) * this.perPage
          };
        },
        result({ data }) {
          this.extrinsics = data.extrinsic;
          if (this.filter) {
            this.totalRows = this.extrinsics.length;
          }
        }
      },
      totalExtrinsics: {
        query: gql`
          subscription total {
            total(where: { name: { _eq: "extrinsics" } }, limit: 1) {
              count
            }
          }
        `,
        result({ data }) {
          if (!this.filter) {
            this.totalRows = data.total[0].count;
          }
        }
      }
    }
  },
  head() {
    return {
      title: this.$t("pages.extrinsics.head_title", {
        networkName: network.name
      }),
      meta: [
        {
          hid: "description",
          name: "description",
          content: this.$t("pages.extrinsics.head_content", {
            networkName: network.name
          })
        }
      ]
    };
  }
};
</script>

<style>
.last-blocks .identicon {
  display: inline-block;
  margin: 0 0.2rem 0 0;
  cursor: copy;
}
</style>
