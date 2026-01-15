# Proyecto-Base-para-la-Actividad-
Proyecto Base para la Actividad


<< ejemplo de caso de prueba en JUnit para addTask >>


import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import java.util.ArrayList;
import java.util.List;

// Clase de prueba
public class TaskManagerTest {

    @Test
    public void testAddTask() {
        // Lista inicial de tareas (simula JSON como lista de objetos)
        List<String> tareas = new ArrayList<>();
        
        // Función que se quiere probar
        addTask(tareas, "Comprar pan");

        // Verifica que la lista ahora tiene una tarea
        assertEquals(1, tareas.size(), "La tarea no se añadió correctamente");

        // Verifica que la tarea añadida es la correcta
        assertEquals("Comprar pan", tareas.get(0), "La tarea añadida no coincide");
    }

    // Método simulado addTask (normalmente estaría en otra clase)
    public void addTask(List<String> tareas, String tarea) {
        tareas.add(tarea);
    }
}
