#!/usr/bin/env python3
"""
TASK MANAGER CLI - Herramienta de gestión de tareas con autenticación remota.

Este script sigue los principios KISS (Keep It Simple) y de responsabilidad única [3, 4].
Uso: python3 task_manager.py -u <usuario> -p <contraseña>
"""

import requests
import sys
import argparse

# --- LÓGICA DE NEGOCIO (Responsabilidad Única) ---

# Lista en memoria para almacenar las tareas durante la ejecución [5]
tasks = []

def add_task():
    """Solicita un título y añade una nueva tarea con estado pendiente."""
    title = input("Ingrese el título de la tarea: ")
    if title.strip():
        tasks.append({"title": title, "status": "pendiente"})
        print(f"Tarea '{title}' agregada con éxito.")
    else:
        print("Error: El título no puede estar vacío.")

def list_tasks():
    """Muestra todas las tareas registradas y su estado actual."""
    if not tasks:
        print("\nNo hay tareas en la lista.")
    else:
        print("\n--- LISTA DE TAREAS ---")
        for i, task in enumerate(tasks, 1):
            print(f"{i}. {task['title']} [{task['status']}]")

def delete_task():
    """Elimina una tarea basada en la coincidencia exacta del título."""
    title = input("Ingrese el título de la tarea a eliminar: ")
    global tasks
    # Filtrado mediante list comprehension para mantener el código limpio [2]
    initial_len = len(tasks)
    tasks = [t for t in tasks if t['title'] != title]
    
    if len(tasks) < initial_len:
        print(f"Tarea '{title}' eliminada.")
    else:
        print(f"No se encontró ninguna tarea con el título: '{title}'")

# --- SISTEMA DE AUTENTICACIÓN ---

def authenticate(expected_user, expected_pass):
    """
    Realiza la autenticación mediante httpbin.org.
    Permite hasta 3 intentos antes de bloquear el acceso [6].
    """
    for intento in range(1, 4):
        print(f"\nAutenticación - Intento {intento} de 3")
        user = input("Usuario: ")
        passwd = input("Contraseña: ")

        # Validamos contra los argumentos pasados al script
        if user == expected_user and passwd == expected_pass:
            try:
                # Uso de la librería 'requests' para llamadas externas [5, 7]
                url = f"https://httpbin.org/basic-auth/{user}/{passwd}"
                response = requests.get(url, auth=(user, passwd), timeout=10)
                
                if response.status_code == 200:
                    print("¡Autenticación exitosa!")
                    return True
            except requests.exceptions.RequestException:
                print("Error de conexión: No se pudo contactar con el servidor.")
        
        print("Credenciales incorrectas.")
    
    return False

# --- FLUJO PRINCIPAL ---

def main():
    # Manejo de parámetros de línea de comandos con argparse [5, 8]
    parser = argparse.ArgumentParser(description="Gestor de Tareas Seguro")
    parser.add_argument("-u", "--user", required=True, help="Nombre de usuario")
    parser.add_argument("-p", "--password", required=True, help="Contraseña del sistema")
    args = parser.parse_args()

    # Inicio del proceso de autenticación interactiva [9]
    if not authenticate(args.user, args.password):
        print("\nAcceso denegado: Se ha superado el número de intentos.")
        sys.exit(1)

    # Menú interactivo
    menu = {
        "1": add_task,
        "2": list_tasks,
        "3": delete_task
    }

    while True:
        print("\n--- MENÚ DE GESTIÓN DE TAREAS ---")
        print("1. Agregar tarea")
        print("2. Listar tareas")
        print("3. Eliminar tarea")
        print("4. Salir")
        
        opcion = input("Elija una opción (1-4): ")

        if opcion == "4":
            print("Saliendo del programa...")
            break
        
        # Ejecución de la función correspondiente mediante el diccionario de acciones
        accion = menu.get(opcion)
        if accion:
            accion()
        else:
            print("Opción no válida, intente de nuevo.")

if __name__ == "__main__":
    # El shebang al inicio permite ejecutarlo como ./task_manager.py en Linux [10, 11]
    main()
