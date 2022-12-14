<template>
  <main>
    <div class="container-fluid min-vh-75">
      <div class="row">
        <div class="col-12">
          <div class="card">
            <div class="card-header">
              <h6>Activity data</h6>
            </div>
            <div class="card-body">

              <button class="btn btn-primary" @click="handleCreate()" v-if="user.role == 'admin'">
                Add new Activity
              </button>
              <data-index
                :columns="columns"
                :actions="action"
                @data-edit="handleEdit"
                @data-crew="handleCrewDeparture"
              >
                <template #sail_date="prop">
                  {{
                    this.$moment(prop.sail_date, "yyyy-mm-DD").format(
                      "DD MMM yyyy"
                    )
                  }}
                </template>
                <template #is_sail="prop">
                  <div v-if="prop.is_sail">
                    <div class="badge badge-sm bg-warning">Sail</div>
                  </div>
                  <div v-else>
                    <div class="badge badge-sm bg-info">Berth</div>
                  </div>
                </template>
              </data-index>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
  <base-modal
    :title="titleModal"
    :modalId="activityModal"
    @on-confirm="isEdit ? acitityEdited() : activityCreated()"
  >
    <template #body>
      <div class="form-group">
        <label for="">Company Name</label>
        <vue-select
          :options="companies"
          label="name"
          v-model="companySelected"
          :value="companySelected"
          class="form-controll"
          :class="{ 'is-invalid': errorState['companyId'] }"
        />
        <div class="invalid-feedback">
          {{ errorState["companyId"] ? errorState["companyId"][0] : "" }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Place of departure</label>
        <vue-select
          :options="harbors"
          label="name"
          class="form-controll"
          v-model="harborSelected"
          :value="harborSelected"
          :class="{ 'is-invalid': errorState['companyId'] }"
        />
        <div class="invalid-feedback">
          {{ errorState["companyId"] ? errorState["companyId"][0] : "" }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Vessel Name</label>
        <vue-select
          :options="vessels"
          label="name"
          v-model="vesselSelected"
          :value="vesselSelected"
          class="form-controll"
          :class="{ 'is-invalid': errorState['vesselId'] }"
        />
        <div class="invalid-feedback">
          {{ errorState["vesselId"] ? errorState["vesselId"][0] : "" }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Sail Date</label>
        <input
          type="date"
          class="form-control"
          :class="{ 'is-invalid': errorState['dateDeparture'] }"
          v-model="departureDate"
        />
        <div class="invalid-feedback">
          {{
            errorState["dateDeparture"] ? errorState["dateDeparture"][0] : ""
          }}
        </div>
      </div>
    </template>
  </base-modal>
  <base-modal title="Data Crew" :modalId="crewModal">
    <template #body>
      <div class="row">
        <div class="col-9">
          <vue-select
            :options="crewExist"
            label="name"
            v-model="crewSelected"
            :value="crewSelected"
            class="form-controll"
            :class="{ 'is-invalid': errorState['company_id'] }"
          />
        </div>
        <div class="col-auto">
          <button class="btn btn-sm btn-primary" @click="addCrewDeparture()">
            Add
          </button>
        </div>
      </div>
      <div class="table-responsive">
        <table class="table">
          <thead>
            <tr>
              <td>#</td>
              <td>Name</td>
              <td></td>
            </tr>
          </thead>
          <tbody v-if="crewDeparture.length > 0">
            <tr v-for="(crew, index) in crewDeparture" :key="index">
              <td>{{ index + 1 }}</td>
              <td>{{ crew.crew.name }}</td>
              <td>
                <button
                  class="btn btn-sm btn-outline-danger"
                  @click="deleteCrew(crew)"
                >
                  <i class="fa fa-trash"></i>
                </button>
              </td>
            </tr>
          </tbody>
          <tbody v-else>
            <tr>
              <td colspan="2" class="text-center text-secondary">
                Empty crew data
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>
    <template #footer>
      <button type="button" class="btn btn-primary" data-bs-dismiss="modal">
        Close
      </button>
    </template>
  </base-modal>
</template>
<script>
import services from "../../services/dataServices";

const columns = [
  {
    name: "Vessel",
    dataIndex: "vessel.name",
  },
  {
    name: "Company",
    dataIndex: "company.name",
  },
  {
    name: "Sail Date",
    dataIndex: "sail_date",
  },
  {
    name: "Total Crew",
    dataIndex: "crew_count",
  },
  {
    name: "Status",
    dataIndex: "is_sail",
  },
];

const action = [
  {
    title: "Edit",
    emit: "data-edit",
    class: "btn-warning",
  },
  {
    title: "Crew",
    emit: "data-crew",
    class: "btn-info",
  },
];
export default {
  data() {
    return {
      titleModal: "",
      activityModal: "activity-modal",
      crewModal: "crew-modal",
      apiPath: "/activity",
      columns,
      isEdit: false,
      id: "",
      dateDeparture: null,
      companies: [],
      companySelected: null,
      harbors: [], 
      harborSelected: null,
      vessels: [],
      vesselSelected: null,
      departureDate: null,
      returnDate: null,
      status: false,
      amount: null,
      crewDeparture: [],
      crewExist: [],
      crewSelected: null,
    };
  },
  computed: {
    errorState() {
      return this.$store.state.data.errorMessage;
    },
    statusLable() {
      return this.status ? "Return" : "Depart";
    },
    user() {
      return this.$store.state.auth.user
    },
    action() {
      return this.user.role == 'admin' ? action : null
    }
  },
  watch: {
    companySelected: async function (next) {
      console.log(next);
      try {
        if (this.companySelected) {
          const result = await services.dataIndex(`/vessel/company/${next.id}`);
          this.vessels = result.data.result;
        } else {
          this.vessels = [];
          this.vesselSelected = null;
        }
      } catch (err) {
        console.log(err);
      }
    },
  },
  mounted() {
    this.initData();
  },
  methods: {
    async initData() {
      this.emitter.emit("fetch", this.apiPath);

      try {
        const harbors = await services.dataIndex("/pelabuhan");
        this.harbors = harbors.data.result;
        const result = await services.dataIndex("/company");
        this.companies = result.data.result;
      } catch (err) {
        console.log(err);
      }
    },
    handleCreate() {
      this.isEdit = false;
      this.titleModal = "Add New Activity";
      this.$store.dispatch("data/validationReset");
      this.emitter.emit(`show-${this.activityModal}`);
    },
    async activityCreated() {
      let data = {
        company_id: this.companySelected?.id ?? null,
        vessel_id: this.vesselSelected?.id ?? null,
        pelabuhan_sail_id: this.harborSelected.id ?? null,
        sail_date: this.departureDate,
      };

      try {
        const result = await this.$store.dispatch("data/create", {
          path: this.apiPath,
          data,
        });
        if (result) {
          this.emitter.emit(`hide-${this.activityModal}`);
          this.emitter.emit("fetch", this.apiPath);
        }
      } catch (err) {
        console.log(err);
      }
    },
    async handleEdit(data) {
      this.titleModal = "Edit  Activity";
      this.isEdit = true;
      this.id = data.id;
      this.$store.dispatch("data/validationReset");
      try {
        const response = await services.dataIndex(`${this.apiPath}/${data.id}`);
        const result = response.data.result;
        this.companySelected = result.company;
        this.vesselSelected = result.vessel;
        this.emitter.emit(`show-${this.activityModal}`);

        result.status == "depart"
          ? (this.status = false)
          : (this.status = true);
        this.amount = result.amount;
        this.departureDate = result.departure_date;
        this.returnDate = result.return_date;
      } catch (err) {
        console.log(err);
      }
    },
    async acitityEdited() {
      let data = {
        companyId: this.companySelected?.id ?? null,
        vesselId: this.vesselSelected?.id ?? null,
        dateDeparture: this.departureDate,
        dateReturn: this.returnDate,
        status: this.status ? "return" : "depart",
        amount: this.amount,
      };

      try {
        const result = await this.$store.dispatch("data/update", {
          path: this.apiPath,
          id: this.id,
          data,
        });
        if (result) {
          this.emitter.emit(`hide-${this.activityModal}`);
          this.emitter.emit("fetch", this.apiPath);
        }
      } catch (err) {
        console.log(err);
      }
    },
    async getDataCrew() {
      const result = await services.dataIndex(`/crew/activity/${this.id}`);
      this.crewExist = result.data.result.crewExists;
      this.crewDeparture = result.data.result.crewDepart;
    },
    handleCrewDeparture(data) {
      this.id = data.id;
      this.getDataCrew();
      this.emitter.emit(`show-${this.crewModal}`);
    },
    async addCrewDeparture() {
      if (!this.crewSelected) {
        return;
      }
      try {
        const data = {
          crew_id: this.crewSelected.id,
          activity_id: this.id,
        };
        await this.$store.dispatch("data/create", {
          path: "/crew/activity",
          data,
        });

        this.crewSelected = null;

        this.getDataCrew();
        this.$store.dispatch("data/index", { path: this.apiPath });
      } catch (err) {
        console.log(err);
      }
    },
    deleteCrew(data) {
      this.id = data.id;
      this.$swal({
        icon: "question",
        title: "Are you sure?",
        html: `Want to delete <strong> ${data.crew.name} </strong>`,
        confirmButtonText: "Yes, delete it!",
      }).then((result) => {
        if (result.isConfirmed) {
          this.crewDeleted(data.id);
        }
      });
    },
    async crewDeleted(id) {
      try {
        await this.$store.dispatch("data/delete", {
          path: `/crew/activity`,
          id,
        });
        this.getDataCrew();
        this.$store.dispatch("data/index", { path: this.apiPath });
      } catch (err) {
        console.log(err);
      }
    },
  },
};
</script>
