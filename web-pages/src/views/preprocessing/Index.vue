<template>
  <div>
    <h2>前処理管理</h2>

    <el-row>
      <kqi-pagination
        v-model="pageStatus"
        :total="total"
        @change="retrieveData"
      />

      <el-col class="right-top-button" :span="8">
        <el-button @click="openPreprocessingDialog">前処理実行</el-button>
        <el-button
          icon="el-icon-edit-outline"
          type="primary"
          plain
          @click="openCreateDialog()"
        >
          新規作成
        </el-button>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col class="search">
        <kqi-smart-search-input
          v-model="searchCondition"
          :configs="searchConfigs"
          @search="search"
        />
      </el-col>
    </el-row>
    <div>
      <el-row>
        <el-table
          ref="table"
          class="data-table pl-index-table"
          :data="preprocessings"
          border
          @row-click="openEditDialog"
        >
          <el-table-column prop="id" label="ID" width="120px" />
          <el-table-column prop="name" label="前処理名" width="auto" />
          <el-table-column
            prop="memo"
            label="メモ"
            width="auto"
            class-name="memo-column"
          />
          <el-table-column prop="createdAt" label="登録日時" width="170px" />
          <el-table-column width="auto">
            <template slot-scope="props">
              <el-button
                icon="el-icon-time"
                type="primary"
                plain
                @click="openHistoryIndex(props.row)"
              >
                履歴
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-row>
      <kqi-pagination
        v-model="pageStatus"
        :total="total"
        @change="retrieveData"
      />
    </div>
    <router-view
      @done="done"
      @cancel="closeDialog"
      @close="closeDialog"
      @copy="handleCopy"
      @return="back"
      @shell="shell"
    />
  </div>
</template>

<script>
import KqiPagination from '@/components/KqiPagination'
import KqiSmartSearchInput from '@/components/KqiSmartSearchInput/Index'
import { createNamespacedHelpers } from 'vuex'
const { mapGetters, mapActions } = createNamespacedHelpers('preprocessing')

export default {
  title: '前処理管理',
  components: {
    KqiPagination,
    KqiSmartSearchInput,
  },
  data() {
    return {
      pageStatus: {
        currentPage: 1,
        currentPageSize: 10,
      },
      searchCondition: {},
      searchConfigs: [
        { prop: 'id', name: 'ID', type: 'number' },
        { prop: 'name', name: '前処理名', type: 'text' },
        { prop: 'memo', name: 'メモ', type: 'text' },
        { prop: 'createdAt', name: '登録日時', type: 'date' },
      ],
      selectedRowId: 0,
    }
  },
  computed: {
    ...mapGetters(['preprocessings', 'total']),
  },
  async created() {
    await this.retrieveData()
  },
  methods: {
    ...mapActions(['fetchPreprocessings']),
    async retrieveData() {
      let params = this.searchCondition
      params.page = this.pageStatus.currentPage
      params.perPage = this.pageStatus.currentPageSize
      params.withTotal = true
      await this.fetchPreprocessings(params)
    },
    async search() {
      this.pageStatus.currentPage = 1
      await this.retrieveData()
    },

    async done(type) {
      if (type === 'delete') {
        // 削除時、表示していたページにデータが無くなっている可能性がある。
        // 総数 % ページサイズ === 1の時、残り1の状態で削除したため、currentPageが1で無ければ1つ前のページに戻す
        if (this.total % this.pageStatus.currentPageSize === 1) {
          if (this.pageStatus.currentPage !== 1) {
            this.pageStatus.currentPage -= 1
          }
        }
      }
      this.closeDialog()
      await this.retrieveData()
      this.showSuccessMessage()
    },
    back() {
      this.$router.go(-1)
    },
    closeDialog() {
      this.$router.push('/preprocessing')
    },
    openCreateDialog() {
      this.$router.push('/preprocessing/create')
    },
    handleCopy(id) {
      this.$router.push('/preprocessing/create/' + id)
    },
    async openHistoryIndex(row) {
      this.$router.push('/preprocessingHistory/' + row.id)
    },

    async openEditDialog(selectedRow) {
      this.selectedRowId = selectedRow.id
      this.$router.push('/preprocessing/edit/' + selectedRow.id)
    },
    openPreprocessingDialog() {
      this.$router.push('preprocessing/run')
    },
    shell(data) {
      this.$router.push(
        '/preprocessing/' + data.id + '/' + data.dataId + '/shell',
      )
    },
  },
}
</script>

<style lang="scss" scoped>
.right-top-button {
  text-align: right;
}

.search {
  text-align: right;
  padding-top: 10px;
}

.el-table /deep/ .memo-column div.cell {
  white-space: pre-wrap;
}

.pagination /deep/ .el-input {
  text-align: left;
  width: 120px;
}

.text {
  font-size: 16px;
}

.item {
  padding: 18px 0;
}

.card-container {
  float: left;
  margin: 20px 20px 10px 0;
  .button {
    padding-top: 30px;
  }
}
</style>
