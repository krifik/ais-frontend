<template>
  <main>
    <div class="container-fluid min-vh-75">
      <div class="row">
        <div class="col-12">
          <div class="card">
            <div class="card-header">
              <h6>Company data</h6>
            </div>
            <div class="card-body">
              <button class="btn btn-primary" @click="createCompany()">
                Add new company
              </button>
              <data-index
                :columns="column"
                :actions="actions"
                @data-edit="updateCompany"
                @data-delete="handleDelete"
                @data-detail="dataDetail"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
  <base-modal
    :title="modalTitle"
    @on-confirm="isEdit ? dataUpdated() : dataCreated()"
  >
    <template #body>
      <h6>Data User</h6>
      <div class="form-group">
        <label for="">Email</label>
        <input
          type="text"
          class="form-control"
          v-model="email"
          :class="{ 'is-invalid': errorState['email'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['email'] ? errorState['email'][0] : '' }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Owner name</label>
        <input
          type="text"
          class="form-control"
          v-model="owner"
          :class="{ 'is-invalid': errorState['owner'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['owner'] ? errorState['owner'][0] : '' }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Password</label>
        <input
          type="text"
          class="form-control"
          v-model="password"
          :class="{ 'is-invalid': errorState['password'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['password'] ? errorState['password'][0] : '' }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Confirm Password</label>
        <input
          type="text"
          class="form-control"
          v-model="cPassword"
          :class="{ 'is-invalid': errorState['cPassword'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['cPassword'] ? errorState['cPassword'][0] : '' }}
        </div>
      </div>
      <h6 class="mt-3">Data Company</h6>
      <div class="form-group">
        <label for="">Company Name</label>
        <input
          type="text"
          class="form-control"
          v-model="name"
          :class="{ 'is-invalid': errorState['name'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['name'] ? errorState['name'][0] : '' }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Registartion Number</label>
        <input
          type="text"
          class="form-control"
          v-model="regNumber"
          :class="{ 'is-invalid': errorState['registration_number'] }"
        />
        <div class="invalid-feedback">
          {{
            errorState['registration_number']
              ? errorState['registration_number'][0]
              : ''
          }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Phone</label>
        <input
          type="text"
          class="form-control"
          v-model="phone"
          :class="{ 'is-invalid': errorState['phone'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['phone'] ? errorState['phone'][0] : '' }}
        </div>
      </div>
      <div class="form-group">
        <label for="">Address</label>
        <input
          type="text"
          class="form-control"
          v-model="address"
          :class="{ 'is-invalid': errorState['address'] }"
        />
        <div class="invalid-feedback">
          {{ errorState['address'] ? errorState['address'][0] : '' }}
        </div>
      </div>
    </template>
  </base-modal>
</template>

<script>
import services from '../../services/dataServices'

const column = [
  {
    name: 'Name',
    dataIndex: 'name',
  },
  {
    name: 'Reg. Number',
    dataIndex: 'registration_number',
  },
  {
    name: 'Phone',
    dataIndex: 'phone',
  },
  {
    name: 'Owner',
    dataIndex: 'user.name',
  },
]

const actions = [
  {
    title: 'Edit',
    emit: 'data-edit',
    class: 'btn-warning',
  },
  {
    title: 'Detail',
    emit: 'data-detail',
    class: 'btn-outline-info',
  },
]

export default {
  data() {
    return {
      apiPath: '/company',
      column,
      actions,
      id: '',
      isEdit: false,
      modalTitle: 'Create new company',
      name: '',
      regNumber: '',
      phone: '',
      address: '',
      owner: '',
      email: '',
      password: '',
      cPassword: '',
    }
  },
  computed: {
    errorState() {
      return this.$store.state.data.errorMessage
    },
  },
  mounted() {
    this.emitter.emit('fetch', this.apiPath)
  },
  methods: {
    resetForm() {
      this.name = ''
      this.regNumber = ''
      this.phone = ''
      this.address = ''
    },
    createCompany() {
      this.isEdit = false
      this.modalTitle = 'Add new company'
      this.emitter.emit('show-modal')
      this.resetForm()
    },
    async updateCompany(data) {
      this.resetForm()
      this.isEdit = true
      this.id = data.id
      try {
        const response = await services.dataIndex(`${this.apiPath}/${data.id}`)
        let result = response.data.result
        this.name = result.name
        this.regNumber = result.registration_number
        this.phone = result.phone
        this.address = result.address
        this.owner = result.user.name
        this.modalTitle = `Update ${data.name}`
        this.emitter.emit('show-modal')
      } catch (err) {
        console.log(err)
      }
    },
    dataUpdated() {
      const data = {
        name: this.name,
        registration_number: this.regNumber,
        phone: this.phone,
        address: this.address,
        owner: this.owner,
      }

      this.$store.dispatch('data/update', {
        path: this.apiPath,
        id: this.id,
        data,
      })

      this.emitter.emit('hide-modal')
      this.emitter.emit('fetch', this.apiPath)
    },
    async dataCreated() {
      let data = {
        name: this.name,
        registration_number: this.regNumber,
        phone: this.phone,
        address: this.address,
        owner: this.owner,
        email: this.email,
        password: this.password,
        cPassword: this.cPassword,
      }

      try {
        this.toggleForm = false
        const result = await this.$store.dispatch('data/create', {
          path: this.apiPath,
          data,
        })
        if (result) {
          this.emitter.emit('hide-modal')
          this.emitter.emit('fetch', this.apiPath)
        }
      } catch (err) {
        console.log(err)
      }
    },
    handleDelete(data) {
      this.id = data.id
      this.$swal({
        icon: 'question',
        title: 'Are you sure?',
        html: `Want to delete <strong> ${data.name} </strong>`,
        confirmButtonText: 'Yes, delete it!',
      }).then((result) => {
        if (result.isConfirmed) {
          this.dataDeleted()
        }
      })
    },
    async dataDeleted() {
      await this.$store.dispatch('data/delete', {
        path: this.apiPath,
        id: this.id,
      })
      this.$store.dispatch('data/index', { path: this.apiPath })
    },
    dataDetail(data) {
      this.$router.push(`/company/${data.id}`)
    }
  },
}
</script>
