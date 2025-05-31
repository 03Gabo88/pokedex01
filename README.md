# pokedex01
"""Proyecto de consulta de pokemon, creando un pokedex practico de principiantes"""
# Pokédex con PokeAPI

"""Este proyecto permite consultar información de cualquier Pokémon usando la API pública pokeapi.co."""

## Requisitos
"""- Python 3.x"""
"""- Bibliotecas: requests, pillow""

## Instalación
""```bash"""
"""pip install requests pillow"""
import requests
import json
import os
from PIL import Image
from urllib.request import urlopen

# Crear carpeta pokedex si no existe
if not os.path.exists('pokedex'):
    os.makedirs('pokedex')

def obtener_datos_pokemon(nombre_pokemon):
    url = f"https://pokeapi.co/api/v2/pokemon/{nombre_pokemon.lower()}"
    respuesta = requests.get(url)
    
    if respuesta.status_code != 200:
        print(f"Error: Pokémon '{nombre_pokemon}' no encontrado.")
        return None
    
    return respuesta.json()

def mostrar_informacion_pokemon(datos):
    print("\nInformación del Pokémon:")
    print(f"Nombre: {datos['name'].capitalize()}")
    print(f"ID: {datos['id']}")
    print(f"Altura: {datos['height'] / 10} m")
    print(f"Peso: {datos['weight'] / 10} kg")
    
    # Tipos
    tipos = [tipo['type']['name'] for tipo in datos['types']]
    print(f"Tipos: {', '.join(tipos)}")
    
    # Habilidades
    habilidades = [habilidad['ability']['name'] for habilidad in datos['abilities']]
    print(f"Habilidades: {', '.join(habilidades)}")
    
    # Estadísticas
    print("\nEstadísticas:")
    for stat in datos['stats']:
        print(f"{stat['stat']['name'].replace('-', ' ').title()}: {stat['base_stat']}")
    
    # Movimientos (mostramos solo los primeros 10 para no saturar)
    movimientos = [movimiento['move']['name'] for movimiento in datos['moves']]
    print(f"\nMovimientos ({len(movimientos)} en total):")
    print(', '.join(movimientos[:10]) + (', ...' if len(movimientos) > 10 else ''))

def mostrar_imagen_pokemon(url_imagen):
    try:
        imagen = Image.open(urlopen(url_imagen))
        imagen.show()
    except:
        print("No se pudo cargar la imagen del Pokémon")

def guardar_en_json(datos, url_imagen):
    pokemon_info = {
        'nombre': datos['name'],
        'id': datos['id'],
        'altura': datos['height'],
        'peso': datos['weight'],
        'tipos': [tipo['type']['name'] for tipo in datos['types']],
        'habilidades': [habilidad['ability']['name'] for habilidad in datos['abilities']],
        'estadisticas': {stat['stat']['name']: stat['base_stat'] for stat in datos['stats']},
        'movimientos': [movimiento['move']['name'] for movimiento in datos['moves']],
        'imagen_url': url_imagen
    }
    
    nombre_archivo = f"pokedex/{datos['name']}.json"
    with open(nombre_archivo, 'w') as archivo:
        json.dump(pokemon_info, archivo, indent=4)
    
    print(f"\nInformación guardada en {nombre_archivo}")

def main():
    print("Bienvenido a la Pokédex")
    
    while True:
        nombre_pokemon = input("\nIngresa el nombre de un Pokémon (o 'salir' para terminar): ").strip()
        
        if nombre_pokemon.lower() == 'salir':
            print("¡Hasta luego!")
            break
            
        datos = obtener_datos_pokemon(nombre_pokemon)
        if datos is None:
            continue
            
        try:
            url_imagen = datos['sprites']['front_default']
            mostrar_informacion_pokemon(datos)
            mostrar_imagen_pokemon(url_imagen)
            guardar_en_json(datos, url_imagen)
        except KeyError as e:
            print(f"Error al procesar los datos del Pokémon: {e}")

if __name__ == "__main__":
    main()
