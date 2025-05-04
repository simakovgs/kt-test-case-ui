<template>
  <div class="form-container">
    <h2>Управление оборудованием</h2>

    <!-- Форма поиска -->
    <div class="search-section">
      <h3>Поиск оборудования</h3>
      <div class="form-group">
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Поиск по серийному номеру или примечанию"
          @keyup.enter="handleSearch"
        >
        <button @click="handleSearch" class="search-btn">Найти</button>
      </div>
    </div>

    <!-- Место для вывода результатов API-запроса -->
    <div class="api-results">
      <h3>Результаты</h3>
      <div v-if="mockResults.length > 0">
        <table class="results-table">
          <thead>
            <tr>
              <th>Тип</th>
              <th>Серийный номер</th>
              <th>Примечание</th>
            </tr>
          </thead>
          <tbody>
            <tr 
              v-for="item in mockResults" 
              :key="item.id"
              @click="selectEquipment(item)"
            >
              <td>{{ item.type_name }}</td>
              <td>{{ item.serial_number }}</td>
              <td>{{ item.note || '-' }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div v-else class="no-results">
        Нет данных для отображения. Выполните поиск.
      </div>
    </div>

    <!-- Форма редактирования -->
    <div class="edit-section" v-if="selectedEquipment">
      <h3>Редактирование записи</h3>
      <form @submit.prevent="handleSave">
        <div class="form-group">
          <label>Тип оборудования:</label>
          <select v-model="form.equipmentTypeId">
            <option 
              v-for="type in equipmentTypes" 
              :key="type.id" 
              :value="type.id"
              :selected="type.id === selectedEquipment.type"
            >
              {{ type.name }} ({{ type.serial_number_mask }})
            </option>
          </select>
        </div>

        <div class="form-group">
          <label>Серийный номер:</label>
          <input 
            type="text" 
            v-model="form.serialNumber" 
          >
        </div>

        <div class="form-group">
          <label>Примечание:</label>
          <textarea 
            v-model="form.notes" 
            rows="3"
          ></textarea>
        </div>

        <div class="action-buttons">
          <button
            type="submit" 
            class="save-btn"
            :disabled="!isFormValid"
          >
            Сохранить
          </button>
          
          <button 
            @click="handleDelete" 
            type="button" 
            class="delete-btn"
            :disabled="!isFormValid"
          >
            Удалить
          </button>
        </div>
      </form>

      <div v-if="result" class="submitted-data">
        <h3>{{ result_message }}</h3>
        <pre>{{ result }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      accessToken: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzQ2ODgxNTYyLCJpYXQiOjE3NDYyNzY3NjIsImp0aSI6IjlhY2JlODVmOTAwYjQ0NDU5MWQyMmQxYTJhZDI2OGMzIiwidXNlcl9pZCI6MX0.ja5VAOxMBkgZT2-TT-Cl_dL4d3a82MPZ8XKJjxUIcWU',
      csrfToken: 'eDb3rEBPUJjRhjcLbWAVWYLvioTyoYTN',
      searchQuery: '',
      selectedEquipment: null,
      editMode: true,
      result: null,
      result_message: null,
      form: {
        id: '',
        equipmentTypeId: '',
        serialNumber: '',
        notes: ''
      },
      equipmentTypes: [],
      mockResults: [],
      apiMessage: '',
      apiError: false
    }
  },

  computed: {
    isFormValid() {
      return (
        this.form.equipmentTypeId && 
        this.form.serialNumber.trim() && 
        this.selectedEquipment
      );
    },
    selectedEquipmentType() {
      if (!this.form.equipmentTypeId) return null;
      return this.equipmentTypes.find(type => type.id === this.form.equipmentTypeId);
    }
  },
  
  methods: {
    async handeSearchBySN() {
      try {
        const responseEquipmentBySN = await fetch(`http://localhost:8000/api/equipment/serial-number/${this.searchQuery}`, {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })

        if (!responseEquipmentBySN.ok) throw new Error(`HTTP error: ${responseEquipmentBySN.status}`)

        const dataEquipmentBySN = await responseEquipmentBySN.json()

        const responseType = await fetch(dataEquipmentBySN.type, {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })

        if (!responseType.ok) throw new Error(`HTTP error: ${responseType.status}`)

        const dataType = await responseType.json()
        
        dataEquipmentBySN.type_name = dataType.name
        dataEquipmentBySN.type = dataType.id

        this.mockResults.push(dataEquipmentBySN)

        const responseEquipmentTypes = await fetch('http://localhost:8000/api/equipment_type/', {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })

        if (!responseEquipmentTypes.ok) throw new Error(`HTTP error: ${responseEquipmentTypes.status}`)

        const dataEquipmentTypes = await responseEquipmentTypes.json()
        this.equipmentTypes = dataEquipmentTypes.results

      } catch (error) {
        console.error('Ошибка отправки:', error)
      } finally {
        this.isSubmitting = false
      }
    },

    async handeSearchByNote() {
      try {
        const responseEquipmentByNote = await fetch(`http://localhost:8000/api/equipment/note/${this.searchQuery}`, {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })

        if (!responseEquipmentByNote.ok) throw new Error(`HTTP error: ${responseEquipmentByNote.status}`)

        const dataEquipmentByNote = await responseEquipmentByNote.json()
        
        for (let i = 0; i < dataEquipmentByNote.length; i++) {
          const data = dataEquipmentByNote[i];
          const responseType = await fetch(data.type, {
            method: 'GET',
            headers: {
              'Authorization': `Bearer ${this.accessToken}`,
              'Accept': 'application/json',
              'Content-Type': 'application/json'
            }
          })

          if (!responseType.ok) throw new Error(`HTTP error: ${responseType.status}`)

          const dataType = await responseType.json()
          
          data.type_name = dataType.name
          data.type = dataType.id

          this.mockResults.push(data);
        }

        const responseEquipmentTypes = await fetch('http://localhost:8000/api/equipment_type/', {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })

        if (!responseEquipmentTypes.ok) throw new Error(`HTTP error: ${responseEquipmentTypes.status}`)

        const dataEquipmentTypes = await responseEquipmentTypes.json()
        this.equipmentTypes = dataEquipmentTypes.results

      } catch (error) {
        console.error('Ошибка отправки:', error)
      } finally {
        this.isSubmitting = false
      }
    },

    async handleSearch() {
      this.mockResults = [];
      await this.handeSearchBySN();
      await this.handeSearchByNote();
    },

    async handleSave() {
      this.result = null
      this.result_message = null
      try {
        const payload = {
          serial_number: this.form.serialNumber,
          type_id: this.form.equipmentTypeId,
          note: this.form.notes
        }

        const response = await fetch(`http://localhost:8000/api/equipment/${this.form.id}/`, {
          method: 'PUT',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Content-Type': 'application/json',
            'X-CSRFTOKEN': this.csrfToken
          },
          body: JSON.stringify(payload)
        })

        const result = await response.json()
        if ("errors" in result) {
          this.result_message = "Ошибка при обновлении";
        } else {
          this.result_message = "Обновление прошло успешно";
        }

        this.result = result
        
      } catch (error) {
        console.error('Ошибка отправки:', error)
      }
    },

    async handleDelete() {
      this.result = null
      this.result_message = null
      try {
        await fetch(`http://localhost:8000/api/equipment/${this.form.id}/`, {
          method: 'DELETE',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Content-Type': 'application/json',
            'X-CSRFTOKEN': this.csrfToken
          },
        })

        this.selectedEquipment = null;
        this.form = {
          id: '',
          equipmentTypeId: '',
          serialNumber: '',
          notes: ''
        };
        await this.handleSearch();
        
      } catch (error) {
        console.error('Ошибка отправки:', error)
      }
    },

    selectEquipment(item) {
      this.selectedEquipment = item;
      this.form = {
        id: this.selectedEquipment.id,
        equipmentTypeId: this.selectedEquipment.type,
        serialNumber: this.selectedEquipment.serial_number,
        notes: this.selectedEquipment.note || ''
      };
    }
  }
}
</script>

<style scoped>
.form-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.search-section,
.edit-section,
.api-results {
  margin-bottom: 30px;
  padding: 15px;
  background-color: #f9f9f9;
  border-radius: 5px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input[type="text"],
select,
textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

textarea {
  min-height: 80px;
  resize: vertical;
}

.action-buttons {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

button {
  padding: 8px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

button:disabled {
  background-color: #6d7c88;
  cursor: not-allowed;
}

.search-btn {
  background-color: #4CAF50;
  color: white;
  margin-left: 10px;
  margin-top: 10px;
}

.edit-btn {
  background-color: #2196F3;
  color: white;
}

.save-btn {
  background-color: #4CAF50;
  color: white;
}

.delete-btn {
  background-color: #f44336;
  color: white;
}

.cancel-btn {
  background-color: #9E9E9E;
  color: white;
}

.small-btn {
  padding: 4px 8px;
  font-size: 12px;
}

.results-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.results-table th,
.results-table td {
  padding: 8px;
  border: 1px solid #ddd;
  text-align: left;
}

.results-table th {
  background-color: #f2f2f2;
}

.results-table tr:hover {
  background-color: #f5f5f5;
}

.message {
  padding: 10px;
  margin-top: 15px;
  border-radius: 4px;
  background-color: #4CAF50;
  color: white;
}

.message.error {
  background-color: #f44336;
}

.no-results {
  padding: 15px;
  text-align: center;
  color: #666;
}

.type-info {
  margin-top: 5px;
  font-size: 0.9em;
  color: #666;
}

.submitted-data {
  margin-top: 30px;
  padding: 15px;
  background-color: #f5f5f5;
  border-radius: 4px;
  text-align: left;
}
</style>