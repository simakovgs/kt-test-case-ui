<template>
  <div class="form-container">
    <h2>Форма добавления оборудования</h2>
    
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <div style="display: flex; gap: 10px; align-items: center; margin-bottom: 10px;">
          <label for="dropdown" style="margin-bottom: 0;">Тип оборудования:</label>
          <button 
            type="button" 
            @click="fetchEquipmentTypes"
            :disabled="isLoading"
            class="load-btn"
          >
            {{ isLoading ? 'Загрузка...' : 'Обновить список' }}
          </button>
        </div>
        <select 
          id="dropdown" 
          v-model="form.equipmentTypeId" 
          :disabled="!dropdownOptions.length"
        >
          <option value="">Выберите тип оборудования</option>
          <option 
            v-for="option in dropdownOptions" 
            :value="option.value" 
            :key="option.value"
          >
            {{ option.text }}
          </option>
        </select>
        <div class="error" v-if="errors.equipmentType">{{ errors.equipmentType }}</div>
      </div>
      
      <div class="form-group">
        <label for="serialNumbers">Серийные номера (по одному в строке):</label>
        <textarea
          id="serialNumbers"
          v-model="form.serialNumbers"
          rows="5"
          placeholder="Введите серийные номера, каждый с новой строки"
          :disabled="!form.equipmentTypeId"
        ></textarea>
        <div class="error" v-if="errors.serialNumbers">{{ errors.serialNumbers }}</div>
        <div v-if="selectedEquipmentMask" class="hint">
          Ожидаемый формат: {{ selectedEquipmentMask }}
        </div>
      </div>

      <div class="form-group">
        <label for="note">Примечание:</label>
        <textarea
          id="note"
          v-model="form.note"
          rows="5"
          placeholder=""
        ></textarea>
      </div>

      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
      </button>
    </form>
    
    <div v-if="result" class="submitted-data">
      <h3>Результат:</h3>
      <pre>{{ result }}</pre>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SerialNumbersForm',
  data() {
    return {
      accessToken: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzQ2ODgxNTYyLCJpYXQiOjE3NDYyNzY3NjIsImp0aSI6IjlhY2JlODVmOTAwYjQ0NDU5MWQyMmQxYTJhZDI2OGMzIiwidXNlcl9pZCI6MX0.ja5VAOxMBkgZT2-TT-Cl_dL4d3a82MPZ8XKJjxUIcWU',
      csrfToken: 'eDb3rEBPUJjRhjcLbWAVWYLvioTyoYTN',
      
      form: {
        equipmentTypeId: '',
        serialNumbers: '',
        note: ''
      },
      equipmentTypes: [],
      errors: {
        equipmentType: '',
        serialNumbers: ''
      },
      isSubmitting: false,
      isLoading: false,
      result: null
    }
  },
  computed: {
    dropdownOptions() {
      return this.equipmentTypes.map(item => ({
        value: item.id,
        text: `${item.name} (Маска: ${item.serial_number_mask})`
      }))
    },
    selectedEquipmentMask() {
      if (!this.form.equipmentTypeId) return ''
      const selected = this.equipmentTypes.find(item => item.id === this.form.equipmentTypeId)
      return selected ? selected.serial_number_mask : ''
    }
  },
  methods: {
    async fetchEquipmentTypes() {
      this.isLoading = true
      try {
        const response = await fetch('http://localhost:8000/api/equipment_type/', {
          method: 'GET',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })

        if (!response.ok) throw new Error(`HTTP error: ${response.status}`)
        
        const data = await response.json()
        this.equipmentTypes = data.results
        
      } catch (error) {
        console.error('Ошибка загрузки типов оборудования:', error)
        this.errors.equipmentType = 'Не удалось загрузить список'
      } finally {
        this.isLoading = false
      }
    },
    
    validateForm() {
      let valid = true
      
      if (!this.form.equipmentTypeId) {
        this.errors.equipmentType = 'Выберите тип оборудования'
        valid = false
      } else {
        this.errors.equipmentType = ''
      }
      
      if (!this.form.serialNumbers.trim()) {
        this.errors.serialNumbers = 'Введите хотя бы один серийный номер'
        valid = false
      } else {
        this.errors.serialNumbers = ''
      }
      
      return valid
    },

    async submitForm() {
      if (!this.validateForm()) return
      
      this.isSubmitting = true
      
      try {
        const serialsArray = this.form.serialNumbers
          .split('\n')
          .map(s => s.trim())
          .filter(s => s)
        
        const payload = []

        for (let i = 0; i < serialsArray.length; i++) {
          const object_to_create = {
            serial_number: serialsArray[i],
            type_id: this.form.equipmentTypeId,
            note: this.form.note
          }
          payload.push(object_to_create)
        }
        
        const response = await fetch('http://localhost:8000/api/equipment/', {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${this.accessToken}`,
            'Content-Type': 'application/json',
            'X-CSRFTOKEN': this.csrfToken
          },
          body: JSON.stringify(payload)
        })
        
        if (!response.ok) throw new Error('Ошибка отправки данных')
        const result = await response.json()
        
        const created_objects_count = result.created_count
        const created_objects_serial_numbers = []
        const errors = []

        for (let i = 0; i < result.created_count; i++) {
          const created_object_serial_number = result.created_objects[i].serial_number
          created_objects_serial_numbers.push(created_object_serial_number)
        }

        for (let i = 0; i < result.errors.length; i++) {
          const error_object_index = result.errors[i].index + 1
          const error_object_data = result.errors[i].errors
          const error_info = {
            'Номер элемента': error_object_index,
            'Информация об ошибке': error_object_data
          }
          errors.push(error_info)
        }

        this.result = {
          'Создано объектов': created_objects_count,
          'Добавленые серийные номера': created_objects_serial_numbers,
          'Ошибки': errors,
        }
                
      } catch (error) {
        console.error('Ошибка отправки:', error)
        alert('Ошибка при отправке данных: ' + error.message)
      } finally {
        this.isSubmitting = false
      }
    }
  },
  mounted() {
    this.fetchEquipmentTypes()
  }
}
</script>

<style scoped>
.form-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

select, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

textarea {
  font-family: monospace;
  min-height: 100px;
}

.error {
  color: #d32f2f;
  font-size: 14px;
  margin-top: 5px;
}

.hint {
  color: #666;
  font-size: 14px;
  margin-top: 5px;
  font-style: italic;
}

button {
  background-color: #4caf50;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #1565c0;
}

button:disabled {
  background-color: #90caf9;
  cursor: not-allowed;
}

.load-btn {
  background-color: #4caf50;
  padding: 8px 12px;
  font-size: 14px;
}

.load-btn:hover {
  background-color: #388e3c;
}

.load-btn:disabled {
  background-color: #81c784;
}

.submitted-data {
  margin-top: 30px;
  padding: 15px;
  background-color: #f5f5f5;
  border-radius: 4px;
  text-align: left;
}

pre {
  white-space: pre-wrap;
  word-wrap: break-word;
}
</style>