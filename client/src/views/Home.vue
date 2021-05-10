<template>
  <div>
    <v-row dense>
      <v-col sm="3" cols="12">
        <v-text-field
          style="width:95%"
          class="pt-2"
          v-model="searchText"
          append-icon="mdi-magnify"
          small
          label="Search"
          clearable
          rounded
          outlined
          dense
          hide-details
        >
        </v-text-field>
      </v-col>
      <v-col sm="3" cols="12">
        <v-select
          style="width:95%"
          class="pt-2"
          v-model="selectedCompany"
          label="Company"
          :items="companyList"
          outlined
          rounded
          dense
          hide-details
        ></v-select>
      </v-col>
      <v-col sm="3" cols="12">
        <v-select
          style="width:95%"
          class="pt-2"
          v-model="selectedGender"
          label="Gender"
          :items="genderList"
          outlined
          rounded
          dense
          hide-details
        ></v-select>
      </v-col>
      <v-col sm="3" cols="12">
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <a-range-picker
              class="pt-3"
              v-on="on"
              v-bind="attrs"
              allowClear
              valueFormat="YYYY-MM-DD"
              v-model="dateRange"
            />
          </template>
          <span>Date Hired</span>
        </v-tooltip>
      </v-col>
      <v-col sm="4" cols="12">
        <v-select
          style="width:95%"
          class="pt-2"
          v-model="selectedDepartment"
          label="Department"
          :items="departmentList"
          outlined
          rounded
          dense
          hide-details
        ></v-select>
      </v-col>
      <v-col sm="4" cols="12">
        <v-select
          style="width:95%"
          class="pt-2"
          v-model="selectedSection"
          label="Section"
          :items="sectionList"
          outlined
          rounded
          dense
          hide-details
        ></v-select>
      </v-col>
      <v-col sm="4" cols="12">
        <v-select
          style="width:95%"
          class="pt-2"
          v-model="selectedTeam"
          label="Team"
          :items="teamList"
          outlined
          rounded
          dense
          hide-details
        ></v-select>
      </v-col>
    </v-row>

    <div class="text-center pt-5" v-if="loading">
      <v-progress-circular indeterminate color="primary"></v-progress-circular>
    </div>
    <v-card
      v-else
      style="overflow-y: scroll"
      :height="$vuetify.breakpoint.height - 220"
    >
      <v-list dense v-for="(item, index) in pageData" :key="index">
        <v-list-item>
          <v-list-item-avatar size="60">
            <v-img
              lazy-src="https://picsum.photos/id/11/10/6"
              :src="`http://adminsql1/photos/${item.EmployeeCode}.jpg`"
            ></v-img>
          </v-list-item-avatar>

          <v-list-item-content>
            <v-list-item-title>
              {{ item.EmployeeCode }} {{ item.EmployeeName }}
            </v-list-item-title>
            <v-list-item-subtitle
              v-text="
                `Department: ${item.DepartmentName} | Section:
              ${item.SectionName} | Team: ${item.TeamName}`
              "
            >
            </v-list-item-subtitle>
          </v-list-item-content>

          <v-list-item-action v-text="`Date Hired: ${item.DateHired}`">
          </v-list-item-action>
        </v-list-item>
      </v-list>
    </v-card>
    <v-pagination
      v-if="!loading"
      v-model="pageNumber"
      :length="pageLength"
      :total-visible="5"
      dark
      prev-icon="mdi-menu-left"
      next-icon="mdi-menu-right"
    ></v-pagination>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
  data() {
    return {
      loading: false,
      pageNumber: 1,
      pageRow: 10,
      searchText: "",
      userInfo: [],
      selectedCompany: "HRD",
      companyList: [
        { text: "HRD Singapore Pte., Ltd.", value: "HRD" },
        { text: "House Technology Industries Pte., Ltd.", value: "HTI" },
        { text: "PV Tech Pte., Ltd.", value: "PV" },
        { text: "WuKong Singapore Pte., Ltd.", value: "WKN" },
        { text: "SCAD SERVICES  Singapore PTE., LTD.", value: "SCD" },
        { text: "Majestic Landscape Corporation", value: "MGTC" },
        { text: "Majestics Energy Corporation", value: "MEC" },
        { text: "Dream Best Academy and Learning", value: "DBA" },
      ],

      selectedDepartment: "ALL",
      departmentList: ["ALL"],
      selectedSection: "ALL",
      sectionList: ["ALL"],
      selectedTeam: "ALL",
      teamList: ["ALL"],

      selectedGender: "ALL",
      genderList: [
        {
          text: "ALL",
          value: "ALL",
        },
        {
          text: "Male",
          value: "M",
        },
        {
          text: "Female",
          value: "F",
        },
      ],

      dateRange: [],
    };
  },
  created() {
    axios.defaults.headers.common["master-api"] =
      "db588403f0a1d3b897442a28724166b4";
    this.loadData();
  },
  watch: {
    selectedCompany() {
      this.pageNumber = 1;
      this.loadData();
    },

    selectedGender() {
      this.pageNumber = 1;
    },
    searchText() {
      this.pageNumber = 1;
    },
    selectedDepartment() {
      this.pageNumber = 1;
      this.loadSection();
    },
    selectedSection() {
      this.pageNumber = 1;
      this.loadTeam();
    },
  },
  methods: {
  
    loadData() {
      this.loading = true;

      const url = `${this.$store.state.api}employees/${this.selectedCompany}`;
      axios
        .get(url)
        .then((res) => {
          this.userInfo = res.data
            .filter((rec) => {
              return rec.EmployeeCode && rec.EmployeeName;
            })
            .filter((rec) => {
              return !rec.SeparationDate;
            })
            .map((rec) => {
              rec.DateHired = moment(rec.DateHired).format("YYYY-MM-DD");
              return rec;
            });
          this.loading = false;
        })
        .then(() => {
          this.loadDepartment();
        });
    },

    loadDepartment() {
      // axios.defaults.headers.common["master-api"] =
      //   "db588403f0a1d3b897442a28724166b4";
      const url = `http://adminsql1/api/company/department/${this.selectedCompany}`;

      this.selectedDepartment = "ALL";
      axios
        .get(url)
        .then((res) => {
          this.departmentList = res.data
            .filter((rec) => {
              return !rec.DeletedDate;
            })
            .map((rec) => {
              return rec.DepartmentName;
            });
        })
        .then(() => {
          this.departmentList.unshift("ALL");
        });
    },

    loadSection() {
      // axios.defaults.headers.common["master-api"] =
      //   "db588403f0a1d3b897442a28724166b4";
      const url = `http://adminsql1/api/company/department/section/${this.selectedCompany}`;

      this.selectedSection = "ALL";
      this.selectedTeam = "ALL";

      axios
        .get(url)
        .then((res) => {
          this.sectionList = res.data
            .filter((rec) => {
              return (
                !rec.DeletedDate &&
                rec.DepartmentName == this.selectedDepartment
              );
            })
            .map((rec) => {
              return rec.SectionName;
            });
        })
        .then(() => {
          this.sectionList.unshift("ALL");
        });
    },
    loadTeam() {
      // axios.defaults.headers.common["master-api"] =
      //   "db588403f0a1d3b897442a28724166b4";
      const url = `http://adminsql1/api/company/department/section/team/${this.selectedCompany}`;
      axios
        .get(url)
        .then((res) => {
          this.teamList = res.data
            .filter((rec) => {
              return (
                !rec.DeletedDate &&
                rec.DepartmentName == this.selectedDepartment &&
                rec.SectionName == this.selectedSection
              );
            })
            .map((rec) => {
              return rec.TeamName;
            });
        })
        .then(() => {
          this.teamList.unshift("ALL");
        });
    },
  },

  computed: {
    dateRangeText() {
      return this.dates.join(" ~ ");
    },
    pageLength() {
      let l = this.computedUsers.length,
        s = this.pageRow;
      return Math.ceil(l / s);
    },
    pageData() {
      const start = (this.pageNumber - 1) * this.pageRow,
        end = start + this.pageRow;
      return this.computedUsers.slice(start, end);
    },

    computedUsers() {
      return this.userInfo
        .filter((rec) => {
          if (this.searchText) {
            return (
              rec.EmployeeCode.toUpperCase().includes(
                this.searchText.toUpperCase()
              ) ||
              rec.EmployeeName.toUpperCase().includes(
                this.searchText.toUpperCase()
              )
            );
          } else {
            return rec;
          }
        })
        .filter((rec) => {
          if (this.selectedGender != "ALL") {
            return rec.Gender == this.selectedGender;
          } else {
            return rec;
          }
        })
        .filter((rec) => {
          if (this.selectedDepartment != "ALL") {
            return rec.DepartmentName == this.selectedDepartment;
          } else {
            return rec;
          }
        })
        .filter((rec) => {
          if (this.selectedSection != "ALL") {
            return rec.SectionName == this.selectedSection;
          } else {
            return rec;
          }
        })
        .filter((rec) => {
          if (this.selectedTeam != "ALL") {
            return rec.TeamName == this.selectedTeam;
          } else {
            return rec;
          }
        })
        .filter((rec) => {
          if (this.dateRange.length > 0) {
            return (
              rec.DateHired >= this.dateRange[0] &&
              rec.DateHired <= this.dateRange[1]
            );
          } else {
            return rec;
          }
        });
    },
  },
};
</script>

<style></style>
