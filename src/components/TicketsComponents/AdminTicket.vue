<template>
  <div class="menu-view">
    <div class="main-container">
      <h1 class="header">Admin Tickets</h1>
      <div class="header-actions">
        <button class="btn" @click="viewUnassigned">Tickets No Asignados</button>
        <button class="btn" @click="viewAssigned">Mis Tickets</button>
      </div>
  
      <!-- Vista de tickets no asignados -->
      <div v-if="currentView === 'unassigned'">
        <h2>Tickets No Asignados</h2>
        <table class="ticket-table">
          <thead>
            <tr>
              <th>Asunto</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ticket in unassignedTickets" :key="ticket.id">
              <td data-label="Asunto">{{ ticket.subject }}</td>
              <td data-label="Acciones">
                <button class="btn btn-update" @click="assignTicket(ticket.id)">Asignar a mí</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
  
      <!-- Vista de tickets asignados -->
      <div v-if="currentView === 'assigned'">
        <h2>Mis Tickets</h2>
        <table class="ticket-table">
          <thead>
            <tr>
              <th>Asunto</th>
              <th>Orden o Paquete</th>
              <th>Tipo de Reclamo</th>
              <th>Asunto</th>
              <th>Estado</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ticket in assignedTickets" :key="ticket.id">
              <td data-label="Asunto">{{ ticket.subject }}</td>
              <td data-label="Orden o Paquete">{{ ticket.order }}</td>
                <td data-label="Tipo de Reclamo">{{ ticket.claimType }}</td>
                <td data-label="Descripción">{{ ticket.description }}</td>
              <td data-label="Estado">{{ ticket.status }}</td>
                
              <td data-label="Acciones">
                <button v-if="ticket.status !== 'closed'" class="btn btn-details" @click="closeTicket(ticket.id)">Cerrar Ticket</button>
                <router-link v-if="ticket.status !== 'closed'" :to="`/update-ticket/${ticket.id}`">
                  <button class="btn btn-update">Actualizar Ticket</button>
                </router-link>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    </div>
  </template>
  
  <script>
  import apiClient from '../../../services/api'
  
  export default {
    data() {
      return {
        unassignedTickets: [],
        assignedTickets: [],
        currentView: 'unassigned', // Estado para controlar la vista
      };
    },
    methods: {
      async viewUnassigned() {
        this.currentView = 'unassigned';
        try {
          const response = await apiClient.get('/unassigned-tickets', {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
          });
          this.unassignedTickets = response.data.map(ticket => ({
          id: ticket.id || ticket.ticket_id,
          order: ticket.order_package,            
          claimType: ticket.claim_type,           
          subject: ticket.subject,                
          description: ticket.description,        
          status: ticket.status,                  
          file: ticket.file,                      
          notifyBy: ticket.notify_by,             
          userId: ticket.user_id,                 
        }));
        } catch (error) {
          console.error('Error al cargar tickets no asignados:', error);
        }
      },
      async viewAssigned() {

        this.currentView = 'assigned';
        try {
          const response = await apiClient.get('/assigned-tickets', {
            headers: {
              Authorization: `Bearer ${localStorage.getItem('token')}`
            }
          });
          this.assignedTickets = response.data.map(ticket => ({
          id: ticket.id || ticket.ticket_id,
          order: ticket.order_package,            
          claimType: ticket.claim_type,           
          subject: ticket.subject,                
          description: ticket.description,        
          status: ticket.status,                  
          file: ticket.file,                      
          notifyBy: ticket.notify_by,             
          userId: ticket.user_id,                 
        }));
        } catch (error) {
          console.error('Error al cargar mis tickets:', error);
        }
      },
      async assignTicket(ticketId) {
    try {
        await apiClient.post(`/assign-ticket/${ticketId}`, {}, {
            headers: {
                Authorization: `Bearer ${localStorage.getItem('token')}`
            }
        });
        this.viewUnassigned(); // Recargar los tickets no asignados
        this.viewAssigned();   // Recargar los tickets asignados
    } catch (error) {
        console.error('Error al asignar el ticket:', error);
    }
},
async closeTicket(ticketId) {
    try {
      await apiClient.put(`/tickets/${ticketId}/close`, {}, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem('token')}`
        }
      });
      // Actualizar la vista de tickets asignados después de cerrar un ticket
      this.viewAssigned();
    } catch (error) {
      console.error('Error al cerrar el ticket:', error);
    }
  }

    },
    mounted() {
      this.viewUnassigned(); // Cargar la vista de tickets no asignados al montar
    }
  };
  </script>
  <style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;500;600;700&display=swap');
  
  .background {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: -1; 
    background-color: #91ebff;
    background: linear-gradient(to top, #e2e2e2, #e8faff);
  }
  
  .main-container {
    max-width: 1000px;
    margin: 40px auto;
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 10px;
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.1);
  }
  
  .header {
    text-align: center;
    margin-bottom: 30px;
  }
  
  .header h2 {
    font-size: 2rem;
    color: #34495e;
    margin-bottom: 10px;
  }
  
  .header-actions {
    display: flex;
    gap: 10px;
    justify-content: center;
    margin-top: 10px;
    flex-wrap: wrap;
  }
  
  .ticket-table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 30px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
  }
  
  .ticket-table th,
  .ticket-table td {
    padding: 15px;
    text-align: left;
    border-bottom: 1px solid #ddd;
  }
  
  .ticket-table th {
    background-color: #00aed5;
    color: white;
    font-weight: bold;
  }
  
  .ticket-table tr:hover {
    background-color: #f1f1f1;
  }
  
  .btn {
    padding: 8px 12px;
    font-size: 1rem;
    background-color: #00aed5;
    color: #fff;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  
  .btn:hover {
    background-color: #2980b9;
  }
  
  .btn-create {
    background-color: #00aed5;
  }
  
  .btn-create:hover {
    background-color: #006c8e;
  }
  
  .btn-update {
    background-color: #27ae60;
  }
  
  .btn-update:hover {
    background-color: #1e8449;
  }
  
  .btn-details {
    background-color: #e74c3c;
  }
  
  .btn-details:hover {
    background-color: #c0392b;
  }
  
  @media (max-width: 768px) {
  .main-container {
    padding: 10px;
  }

  .header h2 {
    font-size: 1.5rem;
  }

  .ticket-table th, .ticket-table td {
    padding: 10px;
    font-size: 0.9rem;
  }

  .btn {
    padding: 6px 10px;
    font-size: 0.9rem;
  }

  .header-actions {
    flex-direction: column;
  }

  .ticket-table thead {
    display: none;
  }

  .ticket-table tr {
    display: block;
    margin-bottom: 20px;
    border: 1px solid #ddd;
    padding: 10px;
    border-radius: 10px;
  }

  .ticket-table td {
    display: flex;
    justify-content: space-between;
    font-size: 0.9rem;
    padding: 5px 10px;
    text-align: right;
  }

  .ticket-table td::before {
    content: attr(data-label);
    font-weight: bold;
    color: #34495e;
    text-align: left;
    margin-right: 10px;
  }
}
  
  
  .loading {
    text-align: center;
    font-size: 1.5rem;
    color: #34495e;
    margin-top: 50px;
  }
  
  .menu-view {
    display: flex;
    flex-direction: column;
    min-height: 66vh;
  }
  </style>