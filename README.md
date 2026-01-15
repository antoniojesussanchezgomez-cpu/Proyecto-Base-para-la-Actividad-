# Proyecto-Base-para-la-Actividad-
Proyecto Base para la Actividad


<< ejemplo de caso de prueba en JUnit para addTask >>

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

class Task {
    String nombre;
    String estado; // "pendiente", "completada", etc.

    Task(String nombre, String estado) {
        this.nombre = nombre;
        this.estado = estado;
    }
}

public class TaskManagerTest {

    // Método simulado addTask
    void addTask(List<Task> tasks, String nombre) {
        tasks.add(new Task(nombre, "pendiente"));
    }

    // Método simulado editTask
    void editTask(List<Task> tasks, int index, String nuevoNombre) {
        tasks.get(index).nombre = nuevoNombre;
    }

    // Método simulado deleteTask
    void deleteTask(List<Task> tasks, int index) {
        tasks.remove(index);
    }

    // Método simulado filterTask
    List<Task> filterTask(List<Task> tasks, String estado) {
        return tasks.stream().filter(t -> t.estado.equals(estado)).collect(Collectors.toList());
    }

    // ------------------ PRUEBAS ------------------

    @Test
    void testAddTask() {
        List<Task> tasks = new ArrayList<>();
        addTask(tasks, "Comprar pan");

        assertEquals(1, tasks.size(), "La tarea no se añadió correctamente");
        assertEquals("Comprar pan", tasks.get(0).nombre, "El nombre de la tarea no coincide");
        assertEquals("pendiente", tasks.get(0).estado, "El estado de la tarea debería ser 'pendiente'");
    }

    @Test
    void testEditTask() {
        List<Task> tasks = new ArrayList<>();
        addTask(tasks, "Comprar pan");

        editTask(tasks, 0, "Comprar leche");
        assertEquals("Comprar leche", tasks.get(0).nombre, "La tarea no se editó correctamente");
    }

    @Test
    void testDeleteTask() {
        List<Task> tasks = new ArrayList<>();
        addTask(tasks, "Comprar pan");
        addTask(tasks, "Llamar al banco");

        deleteTask(tasks, 0);
        assertEquals(1, tasks.size(), "La tarea no se eliminó correctamente");
        assertEquals("Llamar al banco", tasks.get(0).nombre, "La tarea correcta no se mantiene después de eliminar");
    }

    @Test
    void testFilterTask() {
        List<Task> tasks = new ArrayList<>();
        tasks.add(new Task("Comprar pan", "pendiente"));
        tasks.add(new Task("Llamar al banco", "completada"));
        tasks.add(new Task("Hacer ejercicio", "pendiente"));

        List<Task> filtradas = filterTask(tasks, "pendiente");
        assertEquals(2, filtradas.size(), "El filtrado no devolvió la cantidad correcta de tareas");
        for (Task t : filtradas) {
            assertEquals("pendiente", t.estado, "Se devolvió una tarea con estado incorrecto");
        }
    }
}
