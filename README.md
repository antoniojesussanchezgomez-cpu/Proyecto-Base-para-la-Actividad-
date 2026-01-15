# Proyecto-Base-para-la-Actividad-
Proyecto Base para la Actividad


<< ejemplo de caso de prueba en JUnit para addTask >>

✅ Explicación rápida:

Se crea una clase Task para simular cada tarea con nombre y estado.

Los métodos addTask, editTask, deleteTask y filterTask simulan el gestor de tareas.

Cada prueba JUnit verifica que la operación:

Se realiza correctamente.

Mantiene la integridad de la lista de tareas.

Todas las pruebas usan assertEquals para validar resultados automáticamente.
------------------------------------------------------------------------------------------------------------------------------------------------------
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.*;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;

import java.time.LocalDate;
import java.util.List;
import java.util.ArrayList;

/**
 * Suite de pruebas para TaskManager
 * Cubre operaciones CRUD y serialización JSON
 */
class TaskManagerTest {

    private TaskManager taskManager;
    private ObjectMapper objectMapper;

    @BeforeEach
    void setUp() {
        taskManager = new TaskManager();
        objectMapper = new ObjectMapper();
    }

    // ==================== PRUEBAS DE ADDTASK ====================

    @Test
    @DisplayName("Debe añadir correctamente una tarea válida al listado")
    void shouldAddTaskSuccessfully_WhenValidDataProvided() {
        // Arrange
        String title = "Completar proyecto";
        String description = "Finalizar módulo de testing";
        LocalDate dueDate = LocalDate.now().plusDays(7);

        // Act
        taskManager.addTask(title, description, dueDate);
        List<Task> tasks = taskManager.getTasks();

        // Assert
        assertEquals(1, tasks.size(), "Debe haber exactamente una tarea");
        Task addedTask = tasks.get(0);
        assertEquals(title, addedTask.getTitle());
        assertEquals(description, addedTask.getDescription());
        assertEquals(dueDate, addedTask.getDueDate());
    }

    @Test
    @DisplayName("Debe serializar correctamente la tarea a formato JSON")
    void shouldSerializeTaskToJson_WhenTaskAdded() throws Exception {
        // Arrange
        String title = "Revisar código";
        String description = "Code review del PR #123";
        LocalDate dueDate = LocalDate.of(2026, 2, 15);

        // Act
        taskManager.addTask(title, description, dueDate);
        String json = taskManager.toJson();

        // Assert
        assertNotNull(json, "El JSON no debe ser nulo");
        JsonNode jsonNode = objectMapper.readTree(json);
        assertTrue(jsonNode.isArray(), "El JSON debe ser un array");
        assertEquals(1, jsonNode.size());
        
        JsonNode taskNode = jsonNode.get(0);
        assertEquals(title, taskNode.get("title").asText());
        assertEquals(description, taskNode.get("description").asText());
        assertEquals(dueDate.toString(), taskNode.get("dueDate").asText());
    }

    @Test
    @DisplayName("Debe lanzar excepción cuando el título está vacío")
    void shouldThrowException_WhenTitleIsEmpty() {
        // Arrange
        String emptyTitle = "";
        String description = "Descripción válida";
        LocalDate dueDate = LocalDate.now().plusDays(5);

        // Act & Assert
        IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> taskManager.addTask(emptyTitle, description, dueDate),
            "Debe lanzar IllegalArgumentException"
        );
        
        assertTrue(exception.getMessage().contains("título"),
            "El mensaje debe mencionar el título");
    }

    @Test
    @DisplayName("Debe lanzar excepción cuando el título es nulo")
    void shouldThrowException_WhenTitleIsNull() {
        // Arrange
        String description = "Descripción válida";
        LocalDate dueDate = LocalDate.now().plusDays(5);

        // Act & Assert
        assertThrows(
            IllegalArgumentException.class,
            () -> taskManager.addTask(null, description, dueDate)
        );
    }

    @Test
    @DisplayName("Debe lanzar excepción cuando la fecha es pasada")
    void shouldThrowException_WhenDueDateIsInPast() {
        // Arrange
        String title = "Tarea urgente";
        String description = "Esta tarea ya venció";
        LocalDate pastDate = LocalDate.now().minusDays(1);

        // Act & Assert
        IllegalArgumentException exception = assertThrows(
            IllegalArgumentException.class,
            () -> taskManager.addTask(title, description, pastDate),
            "Debe rechazar fechas pasadas"
        );
        
        assertTrue(exception.getMessage().contains("fecha"),
            "El mensaje debe mencionar la fecha");
    }

    @Test
    @DisplayName("Debe aceptar fecha de hoy como válida")
    void shouldAcceptTodayAsValidDueDate() {
        // Arrange
        String title = "Tarea para hoy";
        String description = "Debe completarse hoy";
        LocalDate today = LocalDate.now();

        // Act
        assertDoesNotThrow(() -> taskManager.addTask(title, description, today));

        // Assert
        assertEquals(1, taskManager.getTasks().size());
    }

    @Test
    @DisplayName("Debe permitir descripción vacía")
    void shouldAllowEmptyDescription() {
        // Arrange
        String title = "Tarea sin descripción";
        String emptyDescription = "";
        LocalDate dueDate = LocalDate.now().plusDays(3);

        // Act & Assert
        assertDoesNotThrow(() -> taskManager.addTask(title, emptyDescription, dueDate));
        assertEquals(1, taskManager.getTasks().size());
    }

    // ==================== PRUEBAS DE DELETETASK ====================

    @Test
    @DisplayName("Debe eliminar tarea correctamente por ID")
    void shouldDeleteTask_WhenValidIdProvided() {
        // Arrange
        taskManager.addTask("Tarea 1", "Desc 1", LocalDate.now().plusDays(1));
        taskManager.addTask("Tarea 2", "Desc 2", LocalDate.now().plusDays(2));
        int taskIdToDelete = taskManager.getTasks().get(0).getId();

        // Act
        boolean deleted = taskManager.deleteTask(taskIdToDelete);

        // Assert
        assertTrue(deleted, "Debe retornar true cuando se elimina");
        assertEquals(1, taskManager.getTasks().size());
        assertFalse(taskManager.getTasks().stream()
            .anyMatch(t -> t.getId() == taskIdToDelete));
    }

    @Test
    @DisplayName("Debe retornar false cuando el ID no existe")
    void shouldReturnFalse_WhenDeletingNonExistentTask() {
        // Arrange
        int nonExistentId = 999;

        // Act
        boolean deleted = taskManager.deleteTask(nonExistentId);

        // Assert
        assertFalse(deleted, "Debe retornar false para IDs inexistentes");
    }

    // ==================== PRUEBAS DE UPDATETASK ====================

    @Test
    @DisplayName("Debe actualizar tarea correctamente")
    void shouldUpdateTask_WhenValidDataProvided() {
        // Arrange
        taskManager.addTask("Título original", "Desc original", 
            LocalDate.now().plusDays(5));
        int taskId = taskManager.getTasks().get(0).getId();
        String newTitle = "Título actualizado";
        String newDescription = "Descripción actualizada";
        LocalDate newDate = LocalDate.now().plusDays(10);

        // Act
        boolean updated = taskManager.updateTask(taskId, newTitle, 
            newDescription, newDate);

        // Assert
        assertTrue(updated);
        Task updatedTask = taskManager.getTasks().get(0);
        assertEquals(newTitle, updatedTask.getTitle());
        assertEquals(newDescription, updatedTask.getDescription());
        assertEquals(newDate, updatedTask.getDueDate());
    }

    // ==================== PRUEBAS DE MÚLTIPLES TAREAS ====================

    @Test
    @DisplayName("Debe manejar múltiples tareas correctamente")
    void shouldHandleMultipleTasks() throws Exception {
        // Arrange & Act
        taskManager.addTask("Tarea 1", "Desc 1", LocalDate.now().plusDays(1));
        taskManager.addTask("Tarea 2", "Desc 2", LocalDate.now().plusDays(2));
        taskManager.addTask("Tarea 3", "Desc 3", LocalDate.now().plusDays(3));

        // Assert
        assertEquals(3, taskManager.getTasks().size());
        
        String json = taskManager.toJson();
        JsonNode jsonNode = objectMapper.readTree(json);
        assertEquals(3, jsonNode.size(), "El JSON debe contener 3 tareas");
    }

    @Test
    @DisplayName("Debe retornar array JSON vacío cuando no hay tareas")
    void shouldReturnEmptyJsonArray_WhenNoTasks() throws Exception {
        // Act
        String json = taskManager.toJson();

        // Assert
        JsonNode jsonNode = objectMapper.readTree(json);
        assertTrue(jsonNode.isArray());
        assertEquals(0, jsonNode.size());
    }

    // ==================== PRUEBAS DE VALIDACIÓN DE JSON ====================

    @Test
    @DisplayName("El JSON debe contener todos los campos obligatorios")
    void shouldIncludeAllRequiredFields_InJsonOutput() throws Exception {
        // Arrange
        taskManager.addTask("Tarea completa", "Con todos los campos", 
            LocalDate.of(2026, 3, 20));

        // Act
        String json = taskManager.toJson();
        JsonNode taskNode = objectMapper.readTree(json).get(0);

        // Assert
        assertAll("Verificar campos JSON",
            () -> assertTrue(taskNode.has("id"), "Debe tener campo id"),
            () -> assertTrue(taskNode.has("title"), "Debe tener campo title"),
            () -> assertTrue(taskNode.has("description"), "Debe tener campo description"),
            () -> assertTrue(taskNode.has("dueDate"), "Debe tener campo dueDate"),
            () -> assertTrue(taskNode.has("completed"), "Debe tener campo completed")
        );
    }

    @Test
    @DisplayName("Las nuevas tareas deben marcarse como no completadas por defecto")
    void shouldSetCompletedToFalse_ByDefault() throws Exception {
        // Arrange & Act
        taskManager.addTask("Nueva tarea", "Desc", LocalDate.now().plusDays(1));
        String json = taskManager.toJson();
        JsonNode taskNode = objectMapper.readTree(json).get(0);

        // Assert
        assertFalse(taskNode.get("completed").asBoolean());
    }
}


// ==================== CLASES DE SOPORTE ====================

class Task {
    private static int idCounter = 1;
    private int id;
    private String title;
    private String description;
    private LocalDate dueDate;
    private boolean completed;

    public Task(String title, String description, LocalDate dueDate) {
        this.id = idCounter++;
        this.title = title;
        this.description = description;
        this.dueDate = dueDate;
        this.completed = false;
    }

    // Getters y Setters
    public int getId() { return id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }
    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }
    public boolean isCompleted() { return completed; }
    public void setCompleted(boolean completed) { this.completed = completed; }
}


class TaskManager {
    private List<Task> tasks = new ArrayList<>();
    private ObjectMapper objectMapper = new ObjectMapper();

    public void addTask(String title, String description, LocalDate dueDate) {
        if (title == null || title.trim().isEmpty()) {
            throw new IllegalArgumentException("El título no puede estar vacío");
        }
        if (dueDate.isBefore(LocalDate.now())) {
            throw new IllegalArgumentException("La fecha debe ser hoy o futura");
        }
        tasks.add(new Task(title, description, dueDate));
    }

    public boolean deleteTask(int taskId) {
        return tasks.removeIf(task -> task.getId() == taskId);
    }

    public boolean updateTask(int taskId, String newTitle, 
                             String newDescription, LocalDate newDueDate) {
        for (Task task : tasks) {
            if (task.getId() == taskId) {
                if (newTitle == null || newTitle.trim().isEmpty()) {
                    throw new IllegalArgumentException("El título no puede estar vacío");
                }
                if (newDueDate.isBefore(LocalDate.now())) {
                    throw new IllegalArgumentException("La fecha debe ser hoy o futura");
                }
                task.setTitle(newTitle);
                task.setDescription(newDescription);
                task.setDueDate(newDueDate);
                return true;
            }
        }
        return false;
    }

    public List<Task> getTasks() {
        return new ArrayList<>(tasks);
    }

    public String toJson() {
        try {
            objectMapper.findAndRegisterModules(); // Para LocalDate
            return objectMapper.writeValueAsString(tasks);
        } catch (Exception e) {
            throw new RuntimeException("Error al serializar a JSON", e);
        }
    }
}
