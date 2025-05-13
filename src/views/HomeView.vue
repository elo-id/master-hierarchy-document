<template>
  <div class="home">
    <div class="buttons-container">
      <button @click="addNewTable" class="add-table-btn">
        <img src="@/assets/add.png" class="save-icon">Add New Table
      </button>
      <button class="save-button" @click="isSaveModalVisible = true">
        <img src="@/assets/save.png" class="save-icon">Save
      </button>
    </div>
    <div v-for="(table, index) in tables" :key="index">
      <TableData :tableData="table.data" :documentTitle="table.title" />
    </div>

    <div v-if="isSaveModalVisible" class="alert-modal">
      <div class="modal-alert">
        This action will save the<br />current record!
      </div>
      <div class="alert-modal-buttons">
        <button class="alert-modal-button" @click="proceedSave">
          <span class="alert-modal-button-icon"><img src="@/assets/proceed.png" class="proceed-icon"></span>
          <span class="alert-modal-button-sep">|</span>
          <span>Proceed</span>
        </button>
        <button class="alert-modal-button" @click="cancelSave">
          <span class="alert-modal-button-icon"><img src="@/assets/cancel.png" class="cancel-icon"></span>
          <span class="alert-modal-button-sep">|</span>
          <span>Cancel</span>
        </button>
      </div>
    </div>

    <div v-if="saveSuccessMessage" class="save-success-toast">
      ✅ Record successfully saved.
    </div>
  </div>
</template>

<script>
import TableData from "@/components/TableData.vue";

export default {
  components: { TableData },
  data() {
    return {
      tables: [], // To hold multiple tables with their respective data
      isSaveModalVisible: false,
      saveSuccessMessage: false,
    };
  },
  async created() {
    try {
      const baseURL = window.location.origin + "/ix-DMSDX/plugin/de.elo.ix.plugin.proxy/wf/apps/app/master.hierarchy.documentDashboard";
      const response = await fetch(`${baseURL}/data.json`);
      const rawTables = await response.json();
      console.log(rawTables);
      this.tables = Array.isArray(rawTables)
        ? rawTables.map((t) => ({
            title: t.title || "Document Name", // Default title if not present
            data: t.data || [],
          }))
        : [];
    } catch (error) {
      console.error("Fetch error:", error);
    }
  },
  methods: {
    addNewTable() {
      this.tables.push({
        title: "Document Name",
        data: [], // or appropriate default structure
      });
    },
    proceedSave() {
      this.isSaveModalVisible = false;

      // Simulate saving...
      // e.g., you could save to backend here

      this.saveSuccessMessage = true;
      setTimeout(() => {
        this.saveSuccessMessage = false;
      }, 3000); // hide after 3 seconds
    },
    cancelSave() {
      this.isSaveModalVisible = false;
    },
  },
};
</script>

<style scoped>
.buttons-container {
  display: flex;
  align-items: center;
  margin: 16px 0 0 14px;
  gap: 20px;
}

.add-table-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px 12px 20px;
  background: none;
  color: #0064a0;
  border: 2px solid #0064a0;
  border-radius: 40px;
  cursor: pointer;
  font-size: 14px;
  font-weight: bold;
}

.add-table-btn:hover {
  background-color: #0064a0;
  color: white;
}

.save-button {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px 12px 20px;
  background-color: #0064a0;
  color: white;
  border: 2px solid #0069d1;
  border-radius: 40px;
  cursor: pointer;
  font-size: 14px;
  font-weight: bold;
}

.add-icon, .save-icon, .proceed-icon, .cancel-icon {
  width: 18px;  /* adjust as needed */
  height: 18px;
}

.save-button:hover {
  color: #0064a0;
  background-color: white;
  border: 2px solid #0069d1;
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
  margin: 0 4px 0 0;
  padding: 0;
  font-size: 20px;
  display: inline-block;
  line-height: 1;
}

.alert-modal-button span {
  display: inline-flex;
  align-items: center;
}

.save-success-toast {
  position: fixed; /* key for visibility */
  bottom: 20px;
  right: 20px;
  background-color: #d4edda;
  color: #155724;
  padding: 12px 24px;
  border: 1px solid #c3e6cb;
  border-radius: 6px;
  font-size: 16px;
  z-index: 1000; /* ensure it's on top */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}
</style>
