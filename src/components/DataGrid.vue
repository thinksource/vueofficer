<template>
    <div class="container">
        <h1>Events list:</h1>
      <!-- control bar -->
      <div class="filters">
        <label>Event: 
        <input
          v-model="filterEvent"
          placeholder="Filter by event name..."
          class="filter-input"
          size="17"
          onclick="selectRow(null)"
        /></label>

        <div class="date-filters">
            <label>Date Range: From
            <input
                type="date"
                v-model="startDate"
                class="date-input"
            />
            </label>
            <label>To
            <input
                type="date"
                v-model="endDate"
                class="date-input"
            />
            </label>
        </div>
        <button @click="filterEvent">Search</button>
      </div>
      
  
      <!-- 数据表格区 -->
      <div class="grid-container">
        <table class="data-table">
            <caption>XXX Events Table</caption>
          <thead>
            <tr>
              <th scope="col" v-for="header in headers" :key="header">
                {{ header }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(row, index) in filteredData"
              :key="index"
              @click="selectRow(row)"
              :class="{ selected: selectedRow === row }"
            >
              <td v-for="(value, key) in row" :key="key">
                {{ formatValue(key, value) }}
              </td>
            </tr>
          </tbody>
        </table>
  
        <!-- 详情展示区 -->
        <div class="detail-panel" v-if="selectedRow">
          
          <table class="detail-table">
            <caption class="detail-header">Event Details</caption>
            <tbody>
              <tr v-for="(value, key) in selectedRow" :key="key">
                <td>{{ key.charAt(0).toUpperCase() + key.slice(1) }}</td>
                <td>{{ formatValue(key, value) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted } from 'vue';
  
  // 响应式数据
  const data = ref([]);
  const filterEvent = ref('');
  const startDate = ref('');
  const endDate = ref('');
  let selectedRow = ref(null);
  
  let headers = ref();
  
  // OnMounted hook
  onMounted(async () => {
    try {
      const response = await fetch('/api/test.json');
      data.value = await response.json();
      headers = ref(Object.keys(data.value[0]).map(str=>str.charAt(0).toUpperCase() + str.slice(1)));
      
    } catch (error) {
      console.error('Error loading data:', error);
    }
  });
  
  // 格式化显示内容
  const formatValue = (key, value) => {
    if (key === 'date') {
      return new Date(value).toLocaleDateString();
    }
    if (Array.isArray(value)) {
      return value.join(', ');
    }
    return value;
  };
  
  //
  const formatHeader = (key) => {
    const headerMap = {
      event: 'Event Name',
      date: 'Event Date',
      location: 'Location',
      participants: 'Participants'
    };
    return headerMap[key] || key;
  };
  
  // 过滤逻辑
  const filteredData = computed(() => {
    selectedRow.value = null;
    return data.value.filter(item => {
      // 事件名称过滤
      const nameMatch = item.event.toLowerCase().includes(
        filterEvent.value.toLowerCase()
      );
      
      // 日期范围过滤
      const eventDate = new Date(item.date);
      const start = startDate.value ? new Date(startDate.value) : null;
      const end = endDate.value ? new Date(endDate.value) : null;
      
      let dateMatch = true;
      if (start && end) {
        dateMatch = eventDate >= start && eventDate <= end;
      } else if (start) {
        dateMatch = eventDate >= start;
      } else if (end) {
        dateMatch = eventDate <= end;
      }
      
      return nameMatch && dateMatch;
    });
  });
  
  // 行选择处理
  const selectRow = (row) => {
    selectedRow.value = row;
  };
  </script>
  
  <style scoped>
  .container {
    padding: 20px;
    width: 1280px;
    margin: 0 auto;
    overflow-x: auto;
  }

  .grid-container {
    display: flex;
    height: 100vh;
    overflow-x: auto;
  }
  
  .filters {
    margin-bottom: 20px;
    display: flex;
    gap: 15px;
    align-items: center;
  }
  
  .filter-input {
    padding: 8px 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    flex: 1;
    max-width: 300px;
  }
  
  caption {
    font-weight: bold;
  }
  .date-filters {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  
  .date-input {
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  
  .data-table {
    flex: 3;
    border-collapse: collapse;
    width: 100%;
    border: 1px solid rgba(0,0,0,0.1);
  }
  
  .data-table th,
  .data-table td {
    padding: 12px 15px;
    text-align: left;
    min-width: 150px;
    border: 1px solid #000;
  }
  
  .data-table th {
    background-color: #f5f5f5;
    text-align: center;
  }
  
  .data-table tr:nth-child(even) {
    background-color: #f9f9f9;
    cursor: pointer;
  }
  .data-table tr:nth-child(odd) {
    background-color: #8d8d8d58;
    cursor: pointer;
  }
  
  .data-table tr.selected {
    background-color: #56b8fe;
    font-weight: bold;
  }
  .detail-header {
      font-size: large;
  }
  .detail-panel {
    position: sticky;
    top: 0;
    padding: 5px;
    z-index: 100;
    background-color: #fff;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05);
    padding-right: 10px;
  }
  .detail-panel table {
    width: 100%;
    border-collapse: collapse;
    border: 1px solid #000;
  }
  .detail-panel th,
  .detail-panel td {
    padding: 10px;
    text-align: left;
    border-bottom: 1px solid hsl(0, 0%, 10%);
  }
  .detail-panel td:first-child {
    font-weight: bold;
    font-variant: small-caps;
  }
  .detail-panel td:nth-child(n+2) {
    min-width: 180px;
  }
  .detail-content {
    margin-top: 15px;
  }
  </style>