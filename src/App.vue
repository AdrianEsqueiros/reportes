<template>
  <div>
    <h1 class="text-center text-primary">Reporte General de Denuncias</h1>

    <div class="container">
      <div class="row">
        <!-- <div class="col-md-4">.col-md-4</div> -->
        <div class="d-flex justify-content-center">
          <button
            @click="exportData"
            :class="loading ? 'disabled btn btn-danger' : ' btn btn-success'"
            type="button"
          >
            <span
              v-if="loading"
              class="spinner-border spinner-border-sm"
              role="status"
              aria-hidden="true"
            ></span>
            {{ loading ? 'Downloading ...' : 'Export to CSV' }}
          </button>
        </div>

        <div class="col-lg-12">
          <div class="table-responsive">
            <DataTable
              class="display table table-striped table-bordered"
              id="datatable"
              :columns="columns"
              ajax="http://localhost:3001/api/report/getReport"
              ref="table"
              :options="options"
              cellspacing="0"
              width="99%"
            >
              <thead>
                <tr>
                  <th>Id</th>
                  <th>Usuario</th>
                  <th>Distrito</th>
                  <th>Año</th>
                  <th>Categoria</th>
                  <th>Calificacion</th>
                  <th>Comentario</th>
                  <th>Foto</th>
                  <th>Ubicacion Geografica</th>
                </tr>
              </thead>
              <tbody></tbody>
            </DataTable>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import DataTable from 'datatables.net-vue3'
import Buttons from 'datatables.net-buttons'
import ButtonsHtml5 from 'datatables.net-buttons/js/buttons.html5'
import Select from 'datatables.net-select'
import DataTablesLib from 'datatables.net'
// import 'datatables.net-select'
import axios from 'axios'

DataTable.use(Buttons)
DataTable.use(ButtonsHtml5)
DataTable.use(Select)

export default {
  name: 'DataTableComponent',
  components: { DataTable },
  data() {
    return {
      loading: false,
      options: {
        // dom: 'Bftip',
        select: true,
        serverSide: true,
      },
      columns: [
        { data: 'id' },
        { data: 'user' },
        { data: 'district' },
        { data: 'year' },
        { data: 'category' },
        { data: 'rate' },
        { data: 'comments' },
        {
          data: 'photos',
          render: function (data, type, row) {
            return `<img src="${data}" width="50" height="50" />`
          },
        },
        { data: 'location' },
      ],
      dt: null,
    }
  },
  mounted() {
    this.dt = this.$refs.table.dt()
  },

  methods: {
    async exportData() {
      let data = []
      for (let page = 0; page < 3; page++) {
        try {
          this.loading = true
          console.log('Download in progress')
          const response = await axios.get(
            `http://localhost:3001/api/report/getReport?start=${
              page * 1000000
            }&length=1000000`,
          )
          data = response.data.data

          let csv = ''

          const columns = Object.keys(data[0])
          csv += columns.join(',') + '\n'
          for (const row of data) {
            csv += Object.values(row).join(',') + '\n'
          }
          const blob = new Blob([csv], { type: 'text/csv' })
          const link = document.createElement('a')
          link.href = URL.createObjectURL(blob)
          link.download = `REPORTE-${page + 1}.csv`
          link.click()
          this.loading = false
        } catch (error) {
          this.loading = false
          console.error(error)
        }
      }
    },
  },
}
</script>

<style>
@import 'datatables.net-select-dt';
@import 'datatables.net-buttons-dt';
@import 'datatables.net-dt';
table.dataTable thead {
  background: linear-gradient(to right, #fcb045, #fd1d1d, #833ab4);
  color: white;
}
</style>
