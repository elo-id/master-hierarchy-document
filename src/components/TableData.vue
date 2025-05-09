<template>
  <div class="table-container" v-if="!isTableDeleted">
    <div class="header-row">
      <button class="toggle-btn">
        <div class="main-header-row-menu">
          <span
            :class="isTableVisible ? 'arrow-down' : 'arrow-right'"
            @click="isTableVisible = !isTableVisible"
          ></span>
          <span v-if="!isEditingTitle">{{ documentTitle }}</span>
          <input
            v-else
            v-model="editedTitle"
            @keyup.enter="saveTitle"
            @blur="saveTitle"
            class="title-input"
          />
          <div class="header-actions">
            <button class="edit-title-btn" @click="toggleEditTitle">✎</button>
            <button
              class="delete-table-btn"
              @click="isDeleteTableModalVisible = true"
            >
              ✖
            </button>
          </div>
        </div>

        <div class="search-add-replace">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="🔍︎ Search Document"
            class="search-input"
            @input="applySearchFilter"
          />
          <button class="add-data-btn" @click="openAddDataModal">
            ✚ Add New
          </button>
          <button class="find-replace-btn" @click="openFindReplaceModal">
            ♻ Find & Replace
          </button>
        </div>
      </button>
    </div>

    <div class="separator-line"></div>

    <!-- Delete Table Modal -->
    <div v-if="isDeleteTableModalVisible" class="alert-modal">
      <div class="modal-alert">
        This action will delete the<br />selected record!
      </div>
      <div class="alert-modal-buttons">
        <button class="alert-modal-button" @click="() => proceedDelete()">
          <h1 class="alert-modal-button-icon">🗹</h1>
          <h1 class="alert-modal-button-sep">|</h1>
          Proceed
        </button>
        <button class="alert-modal-button" @click="cancelDelete">
          <h1 class="alert-modal-button-icon">☒</h1>
          <h1 class="alert-modal-button-sep">|</h1>
          Cancel
        </button>
      </div>
    </div>

    <!-- Delete Row Modal -->
    <div v-if="isDeleteRowModalVisible" class="alert-modal">
      <div class="modal-alert">
        This action will delete the<br />selected record!
      </div>
      <div class="alert-modal-buttons">
        <button class="alert-modal-button" @click="removeRow(row)">
          <h1 class="alert-modal-button-icon">🗹</h1>
          <h1 class="alert-modal-button-sep">|</h1>
          Proceed
        </button>
        <button class="alert-modal-button" @click="cancelDelete">
          <h1 class="alert-modal-button-icon">☒</h1>
          <h1 class="alert-modal-button-sep">|</h1>
          Cancel
        </button>
      </div>
    </div>

    <!-- Add Data Modal -->
    <div v-if="isAddDataModalVisible" class="modal">
      <div class="modal-title">✚ | Add Data</div>
      <div class="add-modal-content">
        <div class="input-group-title">
          <label>Document Type:</label>
          <h3 class="not-input">{{ documentTitle }}</h3>
        </div>
        <form @submit.prevent="addRow">
          <div class="input-group">
            <label>Document Type Detail:</label>
            <input
              v-model="newRow.documentType"
              placeholder="Document Type Detail"
              required
            />
          </div>
          <div class="input-group">
            <label>Department:</label>
            <input
              v-model="newRow.department"
              placeholder="Department"
              required
            />
          </div>
          <div class="input-group">
            <label>Plant:</label>
            <input v-model="newRow.plant" placeholder="Plant" required />
          </div>
          <div class="input-group">
            <label>Area:</label>
            <input v-model="newRow.area" placeholder="Area" required />
          </div>
          <div class="input-group">
            <label>Discipline:</label>
            <input v-model="newRow.discipline" placeholder="Discipline" />
          </div>
          <div class="input-group">
            <label>Equipment Type:</label>
            <input
              v-model="newRow.equipmentType"
              placeholder="Equipment Type"
            />
          </div>
          <div class="input-group">
            <label>Equipment Type Detail:</label>
            <input
              v-model="newRow.equipmentTypeDetail"
              placeholder="Equipment Type Detail"
            />
          </div>
          <div class="save-cancel-butt">
            <button class="save-butt" type="submit">🗁 Save</button>
            <button class="cancel-butt" @click="closeAddDataModal">
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Find and Replace Modal -->
    <div v-if="isFindReplaceModalVisible" class="modal">
      <div class="modal-title">♻ | Find and Replace</div>
      <div class="modal-content">
        <form @submit.prevent="findAndReplace">
          <div class="input-group">
            <label>Find:</label>
            <input v-model="findQuery" placeholder="Find" required />
          </div>
          <div class="input-group">
            <label>Replace With:</label>
            <input v-model="replaceQuery" placeholder="Replace with" required />
          </div>
          <div class="button-group">
            <button class="butt-blue" type="button" @click="findAll">
              Find All
            </button>
            <button class="butt-blue" type="button" @click="findNext">
              Find Next
            </button>
            <button class="butt-blue" type="button" @click="replaceCurrent">
              Replace
            </button>
            <button class="butt-blue" type="button" @click="replaceAll">
              Replace All
            </button>
            <button class="butt-white" @click="closeFindReplaceModal">
              Close
            </button>
          </div>
        </form>
      </div>
    </div>

    <div v-if="isTableVisible">
      <div class="table-wrapper">
        <table class="document-table">
          <thead>
            <tr>
              <th>No<br />ㅤ</th>
              <th>
                Document Type Detail<br />
                <select
                  v-model="filterOptions.documentType"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(documentType, index) in uniqueDocumentTypes"
                    :key="index"
                    :value="documentType"
                  >
                    {{ documentType }}
                  </option>
                </select>
              </th>
              <th>
                Department<br />
                <select
                  v-model="filterOptions.department"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(department, index) in uniqueDepartments"
                    :key="index"
                    :value="department"
                  >
                    {{ department }}
                  </option>
                </select>
              </th>
              <th>
                Plant<br />
                <select
                  v-model="filterOptions.plant"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(plant, index) in uniquePlants"
                    :key="index"
                    :value="plant"
                  >
                    {{ plant }}
                  </option>
                </select>
              </th>
              <th>
                Area<br />
                <select
                  v-model="filterOptions.area"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(area, index) in uniqueAreas"
                    :key="index"
                    :value="area"
                  >
                    {{ area }}
                  </option>
                </select>
              </th>
              <th>
                Discipline<br />
                <select
                  v-model="filterOptions.discipline"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(discipline, index) in uniqueDisciplines"
                    :key="index"
                    :value="discipline"
                  >
                    {{ discipline }}
                  </option>
                </select>
              </th>
              <th>
                Equipment Type<br />
                <select
                  v-model="filterOptions.equipmentType"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(type, index) in uniqueEquipmentTypes"
                    :key="index"
                    :value="equipmentType"
                  >
                    {{ equipmentType }}
                  </option>
                </select>
              </th>
              <th>
                Equipment Type Detail<br />
                <select
                  v-model="filterOptions.equipmentTypeDetail"
                  @change="applySearchFilter"
                >
                  <option value="">List Entry ▼</option>
                  <option
                    v-for="(detail, index) in uniqueEquipmentTypeDetails"
                    :key="index"
                    :value="equipmentTypeDetail"
                  >
                    {{ equipmentTypeDetail }}
                  </option>
                </select>
              </th>
              <th>Action<br />ㅤ</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in paginatedTableData" :key="index">
              <td>{{ (currentPage - 1) * rowsPerPage + index + 1 }}</td>
              <td
                v-for="(value, key) in row"
                :key="key"
                :data-match-index="
                  getMatchIndex((currentPage - 1) * rowsPerPage + index, key)
                "
                :class="{
                  highlight: isCurrentMatches(
                    (currentPage - 1) * rowsPerPage + index,
                    key
                  ),
                }"
              >
                {{ value }}
              </td>
              <td>
                <button @click="showDeleteModal(row)" class="action-button">
                  ✖
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="footer-button">
        <div></div>
        <div class="pagination">
          <button
            @click="prevPage"
            :disabled="currentPage === 1"
            class="page-btn"
          >
            &lt;
          </button>
          <button
            v-for="page in totalPages"
            :key="page"
            @click="goToPage(page)"
            :class="['page-btn', { active: page === currentPage }]"
          >
            {{ page }}
          </button>
          <button
            @click="nextPage"
            :disabled="currentPage === totalPages"
            class="page-btn"
          >
            &gt;
          </button>
        </div>
      </div>
    </div>

    <div v-if="deleteSuccessMessage" class="delete-success-toast">
      ❌ Record deleted successfully.
    </div>
  </div>
</template>

<script>
export default {
  props: {
    tableData: {
      type: Array,
      required: true,
    },
    documentTitle: {
      type: String,
      default: "Document Name",
    },
  },
  data() {
    return {
      isTableVisible: true,
      editedTitle: "",
      isEditingTitle: false,
      isTableDeleted: false,
      searchQuery: "",
      searchedTableData: [],
      isDeleteTableModalVisible: false,
      rowToDelete: null,
      isDeleteRowModalVisible: false,
      isAddDataModalVisible: false,
      newRow: {
        documentType: "",
        department: "",
        plant: "",
        area: "",
        discipline: "",
        equipmentType: "",
        equipmentTypeDetail: "",
      },
      isFindReplaceModalVisible: false,
      findQuery: "",
      replaceQuery: "",
      findMatches: [],
      currentMatchIndex: 0,
      filterOptions: {
        documentType: "",
        department: "",
        plant: "",
        area: "",
        discipline: "",
        equipmentType: "",
        equipmentTypeDetail: "",
      },
      currentPage: 1,
      rowsPerPage: 10,
      deleteSuccessMessage: false,
    };
  },
  computed: {
    paginatedTableData() {
      const start = (this.currentPage - 1) * this.rowsPerPage;
      const end = start + this.rowsPerPage;
      return this.searchedTableData.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.searchedTableData.length / this.rowsPerPage);
    },
    uniqueDocumentTypes() {
      return [...new Set(this.tableData.map((row) => row.documentType))];
    },
    uniqueDepartments() {
      return [...new Set(this.tableData.map((row) => row.department))];
    },
    uniquePlants() {
      return [...new Set(this.tableData.map((row) => row.plant))];
    },
    uniqueAreas() {
      return [...new Set(this.tableData.map((row) => row.area))];
    },
    uniqueDisciplines() {
      return [...new Set(this.tableData.map((row) => row.discipline))];
    },
    uniqueEquipmentTypes() {
      return [...new Set(this.tableData.map((row) => row.equipmentType))];
    },
    uniqueEquipmentTypeDetails() {
      return [...new Set(this.tableData.map((row) => row.equipmentTypeDetail))];
    },
  },
  created() {
    this.searchedTableData = this.tableData;
  },
  methods: {
    toggleEditTitle() {
      this.editedTitle = this.documentTitle;
      this.isEditingTitle = true;
      this.$nextTick(() => {
        this.$el.querySelector(".title-input").focus();
      });
    },
    saveTitle() {
      if (this.editedTitle.trim()) {
        this.documentTitle = this.editedTitle.trim();
      }
      this.isEditingTitle = false;
    },
    proceedDelete() {
      this.isTableDeleted = true;
      this.isDeleteTableModalVisible = false;
      this.deleteSuccessMessage = true;

      setTimeout(() => {
        this.deleteSuccessMessage = false;
      }, 3000);
    },
    cancelDelete() {
      this.isDeleteTableModalVisible = false;
      this.isDeleteRowModalVisible = false;
    },
    openAddDataModal() {
      this.isAddDataModalVisible = true;
    },
    closeAddDataModal() {
      this.isAddDataModalVisible = false;
    },
    addRow() {
      const requiredFields = ["documentType", "department", "plant", "area"];
      const valid = requiredFields.every((key) => this.newRow[key].trim());
      if (valid) {
        this.tableData.push({ ...this.newRow });
        this.newRow = {
          documentType: "",
          department: "",
          plant: "",
          area: "",
          discipline: "",
          equipmentType: "",
          equipmentTypeDetail: "",
        };
        this.closeAddDataModal();
      }
    },
    openFindReplaceModal() {
      this.isFindReplaceModalVisible = true;
    },
    closeFindReplaceModal() {
      this.isFindReplaceModalVisible = false;
    },
    applySearchFilter() {
      this.searchedTableData = this.tableData.filter((row) => {
        return Object.values(row).some((value) =>
          value
            ?.toString()
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase())
        );
      });
    },
    findAndReplace() {
      if (this.findQuery.trim() && this.replaceQuery.trim()) {
        this.tableData.forEach((row) => {
          for (let key in row) {
            if (row[key] && row[key].includes(this.findQuery)) {
              row[key] = row[key].replace(
                new RegExp(this.findQuery, "g"),
                this.replaceQuery
              );
            }
          }
        });
        this.closeFindReplaceModal();
      }
    },
    findAll() {
      this.findMatches = [];
      if (!this.findQuery.trim()) return;

      this.tableData.forEach((row, rowIndex) => {
        Object.keys(row).forEach((key) => {
          const value = row[key];
          if (typeof value === "string" && value.includes(this.findQuery)) {
            this.findMatches.push({ rowIndex, key });
          }
        });
      });

      this.currentMatchIndex = 0;
      this.scrollToMatch();
    },
    findNext() {
      if (this.findMatches.length === 0) return;

      this.currentMatchIndex =
        (this.currentMatchIndex + 1) % this.findMatches.length;
      this.scrollToMatch();
    },
    replaceCurrent() {
      if (this.findMatches.length === 0) return;

      const match = this.findMatches[this.currentMatchIndex];
      const row = this.tableData[match.rowIndex];
      const key = match.key;

      row[key] = row[key].replace(this.findQuery, this.replaceQuery);

      this.findAll();
    },
    replaceAll() {
      this.tableData.forEach((row) => {
        Object.keys(row).forEach((key) => {
          const value = row[key];
          if (typeof value === "string" && value.includes(this.findQuery)) {
            row[key] = value.replace(
              new RegExp(this.findQuery, "g"),
              this.replaceQuery
            );
          }
        });
      });
      this.findMatches = [];
      this.currentMatchIndex = 0;
      this.closeFindReplaceModal();
    },
    scrollToMatch() {
      this.$nextTick(() => {
        const matchEl = this.$el.querySelector(
          `[data-match-index="${this.currentMatchIndex}"]`
        );
        if (matchEl) {
          matchEl.scrollIntoView({ behavior: "smooth", block: "center" });
        }
      });
    },
    getMatchIndex(rowIndex, key) {
      return this.findMatches.findIndex(
        (m) => m.rowIndex === rowIndex && m.key === key
      );
    },
    isCurrentMatch(rowIndex, key) {
      const match = this.findMatches[this.currentMatchIndex];
      return match && match.rowIndex === rowIndex && match.key === key;
    },
    filteredTableData() {
      this.filteredTableData = this.tableData.filter((row) => {
        return (
          (this.filterOptions.documentType
            ? row.documentType === this.filterOptions.documentType
            : true) &&
          (this.filterOptions.department
            ? row.department === this.filterOptions.department
            : true) &&
          (this.filterOptions.plant
            ? row.plant === this.filterOptions.plant
            : true) &&
          (this.filterOptions.area
            ? row.area === this.filterOptions.area
            : true) &&
          (this.filterOptions.discipline
            ? row.discipline === this.filterOptions.discipline
            : true) &&
          (this.filterOptions.equipmentType
            ? row.equipmentType === this.filterOptions.equipmentType
            : true) &&
          (this.filterOptions.equipmentTypeDetail
            ? row.equipmentTypeDetail === this.filterOptions.equipmentTypeDetail
            : true)
        );
      });
    },
    getMatchIndex(index, key) {
      return `${index}-${key}`;
    },
    isCurrentMatches(rowIndex, key) {
      if (!this.findMatches.length) return false;
      const current = this.findMatches[this.currentMatchIndex];
      return current && current.rowIndex === rowIndex && current.key === key;
    },
    showDeleteModal(row) {
      this.rowToDelete = row;
      this.isDeleteRowModalVisible = true;
    },
    removeRow() {
      const originalIndex = this.tableData.indexOf(this.rowToDelete);
      if (originalIndex !== -1) {
        this.tableData.splice(originalIndex, 1);

        this.findMatches = this.findMatches.filter(
          (match) => match.rowIndex !== originalIndex
        );

        this.applySearchFilter();
        console.log("Row deleted successfully");
      }

      this.isDeleteRowModalVisible = false;
      this.rowToDelete = null;

      this.deleteSuccessMessage = true;
      setTimeout(() => {
        this.deleteSuccessMessage = false;
      }, 3000); // hide after 3 seconds
    },
    filterTableData() {
      this.searchedTableData = this.tableData.filter((row) => {
        const matchesSearch = Object.values(row).some((val) =>
          String(val).toLowerCase().includes(this.searchQuery.toLowerCase())
        );

        const matchesFilters = Object.keys(this.filterOptions).every((key) => {
          return (
            !this.filterOptions[key] ||
            String(row[key] || "") === this.filterOptions[key]
          );
        });

        return matchesSearch && matchesFilters;
      });
    },
    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) this.currentPage++;
    },
    prevPage() {
      if (this.currentPage > 1) this.currentPage--;
    },
  },
  mounted() {
    this.searchedTableData = this.tableData;
    this.filteredTableData = this.tableData;
  },
  watch: {
    searchQuery: "filterTableData",
    filterOptions: {
      handler: "filterTableData",
      deep: true,
    },
    tableData: {
      immediate: true,
      handler(val) {
        this.searchedTableData = val;
      },
    },
    searchedTableData() {
      this.currentPage = 1;
    },
  },
};
</script>

<style scoped>
.header-row {
  margin-bottom: 10px;
  box-shadow: 0 1px 6px rgba(107, 107, 107, 0.8);
}

.separator-line {
  margin: 12px 0;
  border: 2px solid rgba(107, 107, 107, 0.4);
}

.main-header-row-menu,
.search-add-replace {
  display: flex;
  align-items: center;
  justify-content: center;
}

.search-add-replace {
  gap: 10px;
}

.edit-title-btn {
  font-size: 16px;
  background: none;
  border: none;
  cursor: pointer;
  color: #ffffff;
  border-right: 1px solid white;
}

.edit-title-btn:hover {
  color: #000000;
}

.title-input {
  font-size: 18px;
  font-weight: bold;
  padding: 4px 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.header-actions {
  margin-left: 10px;
  background-color: #0064a0;
  border-radius: 20px;
  padding: 2px 6px;
  display: flex;
  gap: 2px;
}

.delete-table-btn {
  font-size: 16px;
  background: none;
  border: none;
  cursor: pointer;
  color: #ffffff;
}

.delete-table-btn:hover {
  color: #000000;
}

.toggle-btn {
  width: 100%;
  background: rgb(225, 225, 225);
  border: none;
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  padding: 8px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: #000000;
}

.arrow-right::before {
  content: "⯈";
  margin-right: 8px;
  color: #718ebf;
}

.arrow-down::before {
  content: "▼";
  margin-right: 8px;
  color: #718ebf;
}

.search-input {
  padding: 4px;
  font-size: 14px;
  border-radius: 20px;
  border: 1px solid #ccc;
}

.add-data-btn,
.find-replace-btn {
  background-color: #0064a0;
  color: white;
  border: none;
  padding: 4px 16px;
  border-radius: 20px;
  cursor: pointer;
}

.add-data-btn:hover,
.find-replace-btn:hover {
  background-color: #0064a0;
}

.table-container {
  background: none;
  padding: 16px;
  border-radius: 8px;
}

.table-wrapper {
  max-width: 100%;
  overflow-x: auto;
  /* border: 1px solid #cfcfcf; */
}

.document-table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-size: 18px;
  font-family: sans-serif;
}

.document-table thead {
  background-color: #f9fafb;
  color: #00a0dc;
  font-weight: 600;
}

.document-table th,
.document-table td {
  padding: 12px 16px;
  white-space: nowrap;
  border-bottom: 3px solid rgb(141, 141, 141, 0.3);
}

.document-table th:not(:first-child),
.document-table td:not(:first-child) {
  border-left: 3px solid rgb(141, 141, 141, 0.3);
}

.document-table tbody tr:nth-child(odd) {
  background-color: #e2e2e2;
}

.document-table tbody tr:nth-child(even) {
  background-color: #ffffff;
}

.document-table th {
  position: sticky;
  top: 0;
  background-color: #0064a0;
  color: white;
  z-index: 5;
}

.document-table tbody tr:hover {
  background-color: #f0f4ff;
}

select {
  appearance: none;
  color: #8cb2c9;
  background-color: #fff;
  border: 1px solid #8cb2c9;
  padding: 6px 20px;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  transition: border-color 0.3s ease;
  width: 100%;
  text-align: center;
}

select option {
  display: block;
  text-align: center !important;
  text-align-last: center;
  padding-left: 10px;
  width: 100%;
  direction: rtl;
}

select:focus {
  border-color: #007bff;
  outline: none;
  box-shadow: 0 0 3px rgba(0, 123, 255, 0.5);
}

.action-button {
  background-color: #0064a0;
  color: white;
  border: none;
  padding: 2px 20px;
  font-size: 14px;
  border-radius: 50px;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.action-button:hover {
  background-color: #00a0dc;
}

.alert-modal {
  position: fixed;
  display: flex;
  flex-direction: column;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.modal-alert {
  background: white;
  padding: 50px 30px 50px 30px;
  border-radius: 8px 8px 0 0;
  max-width: 600px;
  width: 100%;
  font-size: 30px;
}

.alert-modal-buttons {
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgb(203, 216, 230);
  padding: 40px 30px 40px 30px;
  border-radius: 0 0 8px 8px;
  max-width: 600px;
  width: 100%;
  gap: 40px;
}

.alert-modal-button {
  display: flex;
  align-items: center;
  justify-content: center;
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: white;
  color: #2a5d9f;
  padding: 6px 40px;
  border-radius: 10px;
  font-size: 20px;
}

.alert-modal-button:hover {
  display: flex;
  align-items: center;
  justify-content: center;
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: #2a5d9f;
  color: white;
  padding: 6px 40px;
  border-radius: 10px;
  font-size: 20px;
}

.alert-modal-button-icon,
.alert-modal-button-sep {
  margin: 0 4px 4px 0;
  padding: 0;
  font-size: 20px;
}

.modal {
  position: fixed;
  display: flex;
  flex-direction: column;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.modal-title {
  text-align: start !important;
  background: white;
  padding: 20px 40px 20px 20px;
  border-radius: 8px 8px 0 0;
  max-width: 600px;
  width: 100%;
}

.modal-content {
  background: rgb(203, 216, 230);
  padding: 20px 40px 20px 20px;
  border-radius: 0 0 8px 8px;
  max-width: 600px;
  width: 100%;
}

.add-modal-content {
  background: rgb(203, 216, 230);
  padding: 20px 40px 20px 20px;
  border-radius: 0 0 8px 8px;
  max-width: 600px;
  width: 100%;
}

.input-group-title {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.not-input {
  display: block;
  width: 70%;
  padding: 4px;
  margin: 8px 0;
  border: 1px solid #ccc;
  border-radius: 10px;
  color: #747474;
  background: white;
  font-size: 14px;
  text-align: start;
}

.input-group {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.add-modal-content input {
  display: block;
  width: 70%;
  padding: 4px;
  margin: 8px 0;
  border: 1px solid #ccc;
  border-radius: 10px;
}

.modal-content input {
  display: block;
  width: 75%;
  padding: 4px;
  margin: 8px 0;
  border: 1px solid #ccc;
  border-radius: 10px;
}

.modal-content button {
  margin-top: 10px;
  margin-left: 10px;
}

.save-cancel-butt {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  margin-top: 10px;
  gap: 10px;
}

.save-butt {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: #2a5d9f;
  color: white;
  padding: 10px 40px;
  border-radius: 20px;
}

.save-butt:hover {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: white;
  color: #2a5d9f;
  padding: 10px 40px;
  border-radius: 20px;
}

.cancel-butt {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: white;
  color: #2a5d9f;
  padding: 12px 40px 11px 40px;
  border-radius: 20px;
}

.cancel-butt:hover {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: #2a5d9f;
  color: white;
  padding: 12px 40px 11px 40px;
  border-radius: 20px;
}

.butt-blue {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: #2a5d9f;
  color: white;
  padding: 4px 20px;
  border-radius: 20px;
}

.butt-blue:hover {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: white;
  color: #2a5d9f;
  padding: 4px 20px;
  border-radius: 20px;
}

.butt-white {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: white;
  color: #2a5d9f;
  padding: 4px 30px;
  border-radius: 20px;
}

.butt-white:hover {
  appearance: none;
  border: 1px solid #2a5d9f;
  background-color: #2a5d9f;
  color: white;
  padding: 4px 30px;
  border-radius: 20px;
}

.highlight {
  background-color: #00a0dc;
  font-weight: bold;
}

.footer-button {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 1rem;
}

.pagination {
  display: flex;
  align-items: center;
  gap: 2px;
  padding: 2px 10px;
  border: 1px solid #96b5d6;
  border-radius: 999px;
  width: fit-content;
}

.page-btn {
  width: 30px;
  height: 30px;
  border: 1px solid #96b5d6;
  background-color: white;
  color: #2a5d9f;
  font-size: 14px;
  cursor: pointer;
  text-align: center;
  line-height: 28px;
  border-radius: 4px;
}

.page-btn:hover {
  background-color: #e6f0fa;
}

.page-btn.active {
  background-color: #d1e7fd;
  font-weight: bold;
}

.delete-success-toast {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: #f8d7da; /* soft red */
  color: #721c24; /* dark red text */
  padding: 12px 24px;
  border: 1px solid #f5c6cb;
  border-radius: 6px;
  font-size: 16px;
  z-index: 1000;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}
</style>
