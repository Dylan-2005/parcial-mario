import pygame  # Importa la biblioteca Pygame para crear juegos
import random  # Importa la biblioteca random para generar números aleatorios
import math  # Importa la biblioteca math para funciones matemáticas
from datetime import datetime, timedelta  # Importa clases para manejar fechas y tiempos
import os  # Importa la biblioteca os para interactuar con el sistema operativo

# Dimensiones de pantalla
ANCHO_PANTALLA = 800  # El ancho de la ventana
ALTO_PANTALLA = 600   # El alto de la ventana
Y_MIN = 500           # Altura del suelo (¡el jugador no puede caer más allá de aquí!)

# Configuración del jugador
TIEMPO_INMUNIDAD = 5  # Segundos de inmunidad tras ser golpeado
VELOCIDAD_JUGADOR = 8 # Velocidad lateral del jugador

# Física
GRAVEDAD = 0.5        # Fuerza que tira al jugador hacia abajo
FUERZA_SALTO = -15    # Fuerza hacia arriba al saltar (negativo porque en Pygame el eje Y es invertido)

# Tamaños de las fuentes de texto
FUENTE_TAMANO_VIDAS = 36
FUENTE_TAMANO_GAME_OVER = 74
FUENTE_TAMANO_NIVEL = 48

# Definición de colores en formato RGB
COLOR_FONDO = (135, 206, 235)  # Color de fondo normal
COLOR_FONDO_NOCHE = (25, 25, 112)  # Color de fondo para la noche
COLOR_FONDO_LAVA = (139, 0, 0)  # Color de fondo para la lava
COLOR_FONDO_HIELO = (176, 224, 230)  # Color de fondo para el hielo

# Colores de los objetos en el juego
COLOR_JUGADOR = (0, 0, 255)  # Color del jugador
COLOR_MONEDA = (255, 223, 0)  # Color de las monedas
COLOR_HONGO_CRECE = (255, 0, 0)  # Color del hongo que hace crecer al jugador
COLOR_HONGO_VIDA = (0, 255, 0)  # Color del hongo que da vida al jugador
COLOR_ESTRELLA = (255, 0, 255)  # Color de la estrella que otorga inmunidad

# Colores de los enemigos (Goombas)
COLOR_GOOMBA_CAFE = (139, 69, 19)  # Color del Goomba café
COLOR_GOOMBA_NEGRO = (0, 0, 0)  # Color del Goomba negro
COLOR_GOOMBA_ROJO = (220, 20, 60)  # Color del Goomba rojo
COLOR_GOOMBA_AZUL = (30, 144, 255)  # Color del Goomba azul

# Colores de texto
COLOR_TEXTO = (0, 0, 0)  # Color del texto normal
COLOR_TEXTO_BLANCO = (255, 255, 255)  # Color del texto blanco
COLOR_GAME_OVER = (255, 0, 0)  # Color del texto de Game Over

# Colores del suelo
COLOR_SUELO = (50, 205, 50)  # Color del suelo normal
COLOR_SUELO_ROCA = (105, 105, 105)  # Color del suelo de roca
COLOR_SUELO_LAVA = (255, 69, 0)  # Color del suelo de lava
COLOR_SUELO_HIELO = (240, 248, 255)  # Color del suelo de hielo
COLOR_PLATAFORMA = (139, 69, 19)  # Color de las plataformas

# Definición de los niveles del juego
NIVELES = {
    1: {
        'nombre': 'Llanura Verde',  # Nombre del nivel
        'fondo': COLOR_FONDO,  # Color de fondo del nivel
        'suelo': COLOR_SUELO,  # Color del suelo del nivel
        'texto': COLOR_TEXTO,  # Color del texto en el nivel
        'max_goombas': 2,  # Número máximo de Goombas en el nivel
        'velocidad_goomba': 3,  # Velocidad de los Goombas
        'intervalo_goomba': 4,  # Intervalo de aparición de Goombas
        'tipos_goomba': ['cafe'],  # Tipos de Goombas que aparecen
        'obstaculos': False,  # Si hay obstáculos en el nivel
        'gravedad_mod': 1.0  # Modificador de gravedad para el nivel
    },
    2: {
        'nombre': 'Colinas Rocosas',
        'fondo': (160, 160, 160),
        'suelo': COLOR_SUELO_ROCA,
        'texto': COLOR_TEXTO,
        'max_goombas': 3,
        'velocidad_goomba': 4,
        'intervalo_goomba': 3.5,
        'tipos_goomba': ['cafe', 'negro'],
        'obstaculos': True,
        'gravedad_mod': 1.0
    },
    3: {
        'nombre': 'Noche Oscura',
        'fondo': COLOR_FONDO_NOCHE,
        'suelo': (34, 139, 34),
        'texto': COLOR_TEXTO_BLANCO,
        'max_goombas': 3,
        'velocidad_goomba': 4.5,
        'intervalo_goomba': 3,
        'tipos_goomba': ['negro', 'cafe'],
        'obstaculos': True,
        'gravedad_mod': 1.0
    },
    4: {
        'nombre': 'Caverna de Lava',
        'fondo': COLOR_FONDO_LAVA,
        'suelo': COLOR_SUELO_LAVA,
        'texto': COLOR_TEXTO_BLANCO,
        'max_goombas': 4,
        'velocidad_goomba': 5,
        'intervalo_goomba': 2.5,
        'tipos_goomba': ['rojo', 'negro'],
        'obstaculos': True,
        'gravedad_mod': 1.2
    },
    5: {
        'nombre': 'Mundo de Hielo',
        'fondo': COLOR_FONDO_HIELO,
        'suelo': COLOR_SUELO_HIELO,
        'texto': COLOR_TEXTO,
        'max_goombas': 4,
        'velocidad_goomba': 3,
        'intervalo_goomba': 2,
        'tipos_goomba': ['azul', 'negro'],
        'obstaculos': True,
        'gravedad_mod': 0.8
    },
    6: {
        'nombre': 'Infierno Final',
        'fondo': (64, 0, 0),
        'suelo': (139, 0, 0),
        'texto': COLOR_TEXTO_BLANCO,
        'max_goombas': 5,
        'velocidad_goomba': 6,
        'intervalo_goomba': 1.5,
        'tipos_goomba': ['rojo', 'negro', 'azul'],
        'obstaculos': True,
        'gravedad_mod': 1.3
    }
}

# Clase base para todos los objetos del juego
class Objeto(pygame.sprite.Sprite):
    """Clase base para todos los objetos del juego."""
    def __init__(self, x, y, width=32, height=32):
        super().__init__()  # Inicializa la clase base
        self.image = pygame.Surface((width, height), pygame.SRCALPHA)  # Crea una superficie para el objeto
        self.rect = self.image.get_rect(topleft=(x, y))  # Define el rectángulo del objeto
        self.activo = True  # Indica si el objeto está activo

# Clase para el jugador
class Jugador(pygame.sprite.Sprite):
    """Clase para el jugador."""
    def __init__(self):
        super().__init__()  # Inicializa la clase base
        self.tamano = 'pequeno'  # Tamaño inicial del jugador
        self.image = pygame.Surface((32, 32), pygame.SRCALPHA)  # Crea la imagen del jugador
        self.image.fill(COLOR_JUGADOR)  # Rellena la imagen con el color del jugador
        self.rect = self.image.get_rect(midbottom=(100, Y_MIN))  # Define la posición inicial del jugador
        self.vidas = 3  # Número de vidas del jugador
        self.inmune = False  # Estado de inmunidad del jugador
        self.tiempo_inmunidad = None  # Tiempo restante de inmunidad
        self.monedas_recolectadas = 0  # Contador de monedas recolectadas
        self.velocidad_y = 0  # Velocidad vertical del jugador
        self.en_suelo = True  # Indica si el jugador está en el suelo
        self.nivel = 1  # Nivel actual del jugador
        self.puntos = 0  # Puntos acumulados por el jugador
        self.goombas_derrotados = 0  # Contador de Goombas derrotados

    def mover(self, dx):
        """Mueve al jugador en el eje x."""
        self.rect.x += dx * VELOCIDAD_JUGADOR  # Actualiza la posición horizontal
        self.rect.x = max(0, min(self.rect.x, ANCHO_PANTALLA - self.rect.width))  # Limita el movimiento dentro de la pantalla

    def actualizar_salto(self, gravedad_mod=1.0, plataformas=None):
        """Actualiza la posición del jugador con detección de plataformas."""
        self.velocidad_y += GRAVEDAD * gravedad_mod  # Aplica gravedad
        self.rect.y += self.velocidad_y  # Actualiza la posición vertical
        self.en_suelo = False  # Asume que no está en el suelo

        # Detección de colisiones con plataformas
        if plataformas:
            for plataforma in plataformas:
                if self.rect.colliderect(plataforma.rect):  # Si colisiona con una plataforma
                    if self.velocidad_y > 0:  # Si está cayendo
                        self.rect.bottom = plataforma.rect.top  # Coloca al jugador sobre la plataforma
                        self.velocidad_y = 0  # Detiene la caída
                        self.en_suelo = True  # Indica que está en el suelo
                        return
                    elif self.velocidad_y < 0:  # Si está subiendo
                        self.rect.top = plataforma.rect.bottom  # Coloca al jugador debajo de la plataforma
                        self.velocidad_y = 0  # Detiene el movimiento hacia arriba

        # Detección del suelo
        if self.rect.bottom >= Y_MIN:  # Si el jugador toca el suelo
            self.rect.bottom = Y_MIN  # Coloca al jugador en la posición del suelo
            self.velocidad_y = 0  # Detiene la caída
            self.en_suelo = True  # Indica que está en el suelo

    def saltar(self):
        """Hace que el jugador salte."""
        if self.en_suelo:  # Solo puede saltar si está en el suelo
            self.velocidad_y = FUERZA_SALTO  # Aplica la fuerza de salto
            self.en_suelo = False  # Indica que ya no está en el suelo

    def crecer(self):
        """Cambia el tamaño del jugador a grande."""
        if self.tamano == 'pequeno':  # Si el jugador es pequeño
            self.tamano = 'grande'  # Cambia el tamaño a grande
            old_bottom = self.rect.bottom  # Guarda la posición inferior
            old_centerx = self.rect.centerx  # Guarda la posición central
            self.image = pygame.Surface((32, 64), pygame.SRCALPHA)  # Crea una nueva imagen más alta
            self.image.fill(COLOR_JUGADOR)  # Rellena la nueva imagen con el color del jugador
            self.rect = self.image.get_rect()  # Actualiza el rectángulo de la imagen
            self.rect.bottom = old_bottom  # Restaura la posición inferior
            self.rect.centerx = old_centerx  # Restaura la posición central

    def reducir(self):
        """Cambia el tamaño del jugador a pequeño."""
        if self.tamano == 'grande':  # Si el jugador es grande
            self.tamano = 'pequeno'  # Cambia el tamaño a pequeño
            old_bottom = self.rect.bottom  # Guarda la posición inferior
            old_centerx = self.rect.centerx  # Guarda la posición central
            self.image = pygame.Surface((32, 32), pygame.SRCALPHA)  # Crea una nueva imagen más pequeña
            self.image.fill(COLOR_JUGADOR)  # Rellena la nueva imagen con el color del jugador
            self.rect = self.image.get_rect()  # Actualiza el rectángulo de la imagen
            self.rect.bottom = old_bottom  # Restaura la posición inferior
            self.rect.centerx = old_centerx  # Restaura la posición central

    def activar_inmunidad(self):
        """Activa la inmunidad del jugador."""
        self.inmune = True  # Cambia el estado a inmune
        self.tiempo_inmunidad = datetime.now() + timedelta(seconds=TIEMPO_INMUNIDAD)  # Establece el tiempo de inmunidad

    def actualizar_inmunidad(self):
        """Actualiza el estado de inmunidad."""
        if self.inmune:  # Si el jugador es inmune
            if datetime.now() > self.tiempo_inmunidad:  # Si el tiempo de inmunidad ha pasado
                self.inmune = False  # Cambia el estado a no inmune
                self.image.set_alpha(255)  # Restaura la opacidad de la imagen
    def subir_nivel(self):
        """Sube al siguiente nivel."""
        if self.nivel < len(NIVELES):
            self.nivel += 1
            self.goombas_derrotados = 0  # Reinicia el contador de Goombas
            self.puntos += 500 * self.nivel  # Premio por subir de nivel
            return True  # Nivel subido exitosamente
        return False  # Ya está en el último nivel

# Clase para las monedas del juego
class Moneda(Objeto):
    """Clase para las monedas del juego."""
    def __init__(self, plataformas=None):
        # Decide si la moneda aparecerá en una plataforma o en el aire
        if plataformas and random.random() > 0.6:  # 60% de probabilidad de aparecer en plataformas
            plataforma = random.choice(plataformas.sprites())  # Elige una plataforma al azar
            x = random.randint(plataforma.rect.left + 10, plataforma.rect.right - 42)  # Posición horizontal aleatoria
            y = plataforma.rect.top - 32  # Posición vertical justo encima de la plataforma
        else:
            x = random.randint(0, ANCHO_PANTALLA - 32)  # Posición horizontal aleatoria en la pantalla
            y = random.randint(100, 400)  # Posición vertical aleatoria

        super().__init__(x, y)  # Inicializa la clase base con la posición
        self.image.fill(COLOR_MONEDA)  # Rellena la imagen de la moneda con su color
        self.tiempo_animacion = 0  # Inicializa el tiempo de animación

    def update(self):
        """Actualiza la animación de la moneda."""
        self.tiempo_animacion += 0.2  # Incrementa el tiempo de animación
        offset = int(math.sin(self.tiempo_animacion) * 5)  # Calcula el desplazamiento vertical
        self.rect.y += offset  # Aplica el desplazamiento a la posición vertical de la moneda

# Clase para los hongos que otorgan habilidades al jugador
class Hongo(Objeto):
    """Clase para los hongos que otorgan habilidades al jugador."""
    def __init__(self, x, y, tipo):
        super().__init__(x, y)  # Inicializa la clase base con la posición
        self.tipo = tipo  # Guarda el tipo de hongo
        color = COLOR_HONGO_CRECE if tipo == 'crecimiento' else COLOR_HONGO_VIDA  # Define el color según el tipo
        self.image.fill(color)  # Rellena la imagen del hongo con su color
        self.tiempo_reactivacion = None  # Inicializa el tiempo de reactivación

    def desactivar(self):
        """Desactiva el hongo y establece un tiempo de reactivación."""
        self.activo = False  # Cambia el estado a inactivo
        self.tiempo_reactivacion = datetime.now() + timedelta(seconds=15)  # Establece el tiempo de reactivación
        self.image.set_alpha(100)  # Reduce la opacidad de la imagen

    def activar(self):
        """Reactiva el hongo."""
        self.activo = True  # Cambia el estado a activo
        self.image.set_alpha(255)  # Restaura la opacidad de la imagen

# Clase para la estrella que otorga inmunidad al jugador
class Estrella(Objeto):
    """Clase para la estrella que otorga inmunidad al jugador."""
    def __init__(self):
        x = random.randint(0, ANCHO_PANTALLA - 32)  # Posición horizontal aleatoria
        y = random.randint(100, 400)  # Posición vertical aleatoria
        super().__init__(x, y)  # Inicializa la clase base con la posición
        self.image.fill(COLOR_ESTRELLA)  # Rellena la imagen de la estrella con su color
        self.tiempo_reactivacion = None  # Inicializa el tiempo de reactivación
        self.tiempo_animacion = 0  # Inicializa el tiempo de animación

    def desactivar(self):
        """Desactiva la estrella temporalmente."""
        self.activo = False  # Cambia el estado a inactivo
        self.tiempo_reactivacion = datetime.now() + timedelta(seconds=20)  # Establece el tiempo de reactivación
        self.rect.topleft = (-100, -100)  # Mueve la estrella fuera de la pantalla

    def resetear(self):
        """Reinicia la posición de la estrella."""
        self.activo = True  # Cambia el estado a activo
        self.image.set_alpha(255)  # Restaura la opacidad de la imagen
        self.rect.topleft = (random.randint(0, ANCHO_PANTALLA - 32), random.randint(100, 400))  # Posición aleatoria

# Clase para los Goombas (enemigos)
class Goomba(pygame.sprite.Sprite):
    def __init__(self, tipo='cafe', velocidad=3):
        super().__init__()  # Inicializa la clase base
        self.tipo = tipo  # Guarda el tipo de Goomba
        self.image = pygame.Surface((32, 32))  # Crea la imagen del Goomba
        colores = {
            'cafe': COLOR_GOOMBA_CAFE,
            'negro': COLOR_GOOMBA_NEGRO,
            'rojo': COLOR_GOOMBA_ROJO,
            'azul': COLOR_GOOMBA_AZUL
        }
        self.image.fill(colores.get(tipo, COLOR_GOOMBA_CAFE))  # Rellena la imagen con el color correspondiente
        self.rect = self.image.get_rect(bottomright=(ANCHO_PANTALLA, Y_MIN))  # Define la posición inicial
        self.velocidad = velocidad  # Establece la velocidad del Goomba
        self.direccion = -1  # Dirección inicial del movimiento
        self.vidas = 2 if tipo in ['rojo', 'azul'] else 1  # Define el número de vidas según el tipo
        self.tiempo_animacion = 0  # Inicializa el tiempo de animación

    def update(self):
        """Actualiza el movimiento y animación del Goomba."""
        # Movimiento horizontal
        self.rect.x += self.direccion * abs(self.velocidad)  # Mueve el Goomba en la dirección establecida
        # Rebote en los bordes
        if self.rect.left <= 0 or self.rect.right >= ANCHO_PANTALLA:  # Si toca los bordes de la pantalla
            self.direccion *= -1  # Cambia la dirección
        # Animación vertical
        self.tiempo_animacion += 0.1  # Incrementa el tiempo de animación
        offset_y = int(math.sin(self.tiempo_animacion * 2) * 2)  # Calcula el desplazamiento vertical
        self.rect.y = Y_MIN + offset_y - self.rect.height  # Aplica el desplazamiento a la posición vertical

    def golpear(self):
        """Reduce las vidas del Goomba."""
        self.vidas -= 1  # Resta una vida
        if self.vidas <= 0:  # Si no tiene más vidas
            self.kill()  # Elimina el Goomba
            return True  # Indica que ha sido derrotado
        else:
            self.image.set_alpha(150)  # Reduce la opacidad de la imagen
            return False  # Indica que sigue vivo

# Clase para las plataformas
class Plataforma(Objeto):
    """Plataformas flotantes con diseño mejorado."""
    def __init__(self, x, y, width=120):
        super().__init__(x, y, width, 24)  # Inicializa la clase base con la posición y tamaño de la plataforma
        self.image.fill(COLOR_PLATAFORMA)  # Rellena la imagen de la plataforma con su color
        # Añadir textura a la plataforma
        for i in range(0, width, 32):  # Crea una textura en la plataforma
            pygame.draw.rect(self.image, (100, 50, 20), (i, 0, 28, 24))  # Dibuja rectángulos en la plataforma

class Obstaculo(Objeto):
    """Obstáculos estáticos en el suelo."""
    def __init__(self, x, y):
        super().__init__(x, y, 32, 64)  # Inicializa la clase base con la posición y tamaño del obstáculo
        self.image.fill((139, 69, 19))  # Rellena la imagen del obstáculo con un color marrón

def cargar_sonidos():
    """Carga los sonidos del juego de forma segura."""
    sonidos = {}  # Diccionario para almacenar los sonidos
    archivos_sonido = {
        'moneda': 'assets/sounds/coin.wav',  # Ruta del sonido de la moneda
        'salto': 'assets/sounds/jump.wav',  # Ruta del sonido del salto
        'powerup': 'assets/sounds/powerup.wav',  # Ruta del sonido de poder
        'game_over': 'assets/sounds/game_over.wav',  # Ruta del sonido de Game Over
        'hit': 'assets/sounds/hit.wav',  # Ruta del sonido de golpe
        'nivel_up': 'assets/sounds/level_up.wav'  # Ruta del sonido de subir de nivel
    }

    for nombre, archivo in archivos_sonido.items():
        try:
            if os.path.exists(archivo):  # Verifica si el archivo de sonido existe
                sonidos[nombre] = pygame.mixer.Sound(archivo)  # Carga el sonido
            else:
                sonidos[nombre] = pygame.mixer.Sound(buffer=b'\x00' * 1024)  # Crea un sonido vacío si no existe
        except pygame.error:
            sonidos[nombre] = pygame.mixer.Sound(buffer=b'\x00' * 1024)  # Maneja errores de carga de sonido
    return sonidos  # Devuelve el diccionario de sonidos

def crear_plataformas(nivel):
    """Crea plataformas según el nivel."""
    plataformas = pygame.sprite.Group()  # Grupo para almacenar las plataformas

    # Define las plataformas para cada nivel
    if nivel == 1:
        plataformas.add(Plataforma(150, Y_MIN - 100, 100))  # Añade una plataforma al nivel 1
    elif nivel == 2:
        plataformas.add(Plataforma(100, Y_MIN - 150, 80))  # Añade plataformas al nivel 2
        plataformas.add(Plataforma(400, Y_MIN - 200, 120))
    elif nivel == 3:
        for x in range(100, 800, 200):  # Añade plataformas en el nivel 3
            plataformas.add(Plataforma(x, Y_MIN - 150, 90))
    elif nivel == 4:
        plataformas.add(Plataforma(200, Y_MIN - 300, 100))  # Añade plataformas al nivel 4
        plataformas.add(Plataforma(500, Y_MIN - 250, 100))
    elif nivel == 5:
        plataformas.add(Plataforma(100, Y_MIN - 180, 60))  # Añade plataformas al nivel 5
        plataformas.add(Plataforma(300, Y_MIN - 300, 120))
        plataformas.add(Plataforma(600, Y_MIN - 220, 100))
    elif nivel == 6:
        for i in range(5):  # Añade plataformas en el nivel 6
            plataformas.add(Plataforma(100 + i * 140, Y_MIN - (100 + i * 40), 80))
    return plataformas  # Devuelve el grupo de plataformas

def crear_obstaculos(nivel):
    """Crea obstáculos según el nivel."""
    obstaculos = pygame.sprite.Group()  # Grupo para almacenar los obstáculos
    if NIVELES[nivel]['obstaculos']:  # Verifica si el nivel tiene obstáculos
        num_obstaculos = min(3, nivel - 1)  # Define el número de obstáculos
        for _ in range(num_obstaculos):
            x = random.randint(200, ANCHO_PANTALLA - 200)  # Posición horizontal aleatoria
            obstaculo = Obstaculo(x, Y_MIN - 64)  # Crea un nuevo obstáculo
            obstaculos.add(obstaculo)  # Añade el obstáculo al grupo
    return obstaculos  # Devuelve el grupo de obstáculos

def reiniciar_nivel(jugador):
    """Reinicia el nivel actual con todos los objetos."""
    todos_sprites = pygame.sprite.Group(jugador)  # Grupo para almacenar todos los sprites
    plataformas = crear_plataformas(jugador.nivel)  # Crea plataformas para el nivel actual

    # Generar Goombas desde el inicio del nivel
    goombas = pygame.sprite.Group()  # Grupo para almacenar los Goombas
    for i in range(NIVELES[jugador.nivel]['max_goombas']):  # Genera Goombas según el nivel
        tipo_goomba = random.choice(NIVELES[jugador.nivel]['tipos_goomba'])  # Elige un tipo de Goomba
        x = random.randint(100, ANCHO_PANTALLA - 100)  # Posición horizontal aleatoria
        goomba = Goomba(tipo_goomba, NIVELES[jugador.nivel]['velocidad_goomba'])  # Crea un nuevo Goomba
        goomba.rect.bottom = Y_MIN  # Coloca el Goomba en la parte inferior
        goomba.direccion = random.choice([-1, 1])  # Asigna una dirección aleatoria
        goombas.add(goomba)  # Añade el Goomba al grupo
        todos_sprites.add(goomba)  # Añade el Goomba al grupo de todos los sprites

    # Generar monedas asociadas a plataformas
    monedas = pygame.sprite.Group()  # Grupo para almacenar las monedas
    for _ in range(8 + jugador.nivel):  # Genera monedas según el nivel
        moneda = Moneda(plataformas)  # Crea una nueva moneda
        monedas.add(moneda)  # Añade la moneda al grupo

    # Posicionar hongos en plataformas o en el suelo
    hongos = pygame.sprite.Group()  # Grupo para almacenar los hongos
    if plataformas:  # Si hay plataformas
        for plataforma in plataformas.sprites()[:2]:  # Solo en las primeras 2 plataformas
            if random.random() > 0.5:  # 50% de probabilidad de generar un hongo
                tipo = 'crecimiento' if random.random() > 0.5 else 'vida'  # Elige el tipo de hongo
                hongo = Hongo(plataforma.rect.centerx, plataforma.rect.top - 32, tipo)  # Crea un nuevo hongo
                hongos.add(hongo)  # Añade el hongo al grupo
    else:
        # Si no hay plataformas, colocar en el suelo
        hongos.add(Hongo(200, Y_MIN - 32, 'crecimiento'))  # Crea un hongo de crecimiento
        hongos.add(Hongo(600, Y_MIN - 32, 'vida'))  # Crea un hongo de vida

    estrella = Estrella()  # Crea una estrella
    obstaculos = crear_obstaculos(jugador.nivel)  # Crea obstáculos para el nivel

    # Añade todos los objetos al grupo de sprites
    todos_sprites.add(plataformas, monedas, hongos, estrella, obstaculos)
    return (
        todos_sprites,
        monedas,
        hongos,
        estrella,
        obstaculos,
        plataformas,
        goombas
    )

def mostrar_transicion_nivel(ventana, nivel, fuente_nivel):
    """Muestra la transición entre niveles."""
    overlay = pygame.Surface((ANCHO_PANTALLA, ALTO_PANTALLA))  # Crea una superficie para la transición
    overlay.set_alpha(200)  # Establece la transparencia
    overlay.fill((0, 0, 0))  # Rellena la superficie con negro
    ventana.blit(overlay, (0, 0))  # Dibuja la superficie en la ventana

    # Muestra el texto del nivel
    texto_nivel = fuente_nivel.render(f'NIVEL {nivel}', True, (255, 255, 255))  # Renderiza el texto del nivel
    nombre_nivel = fuente_nivel.render(NIVELES[nivel]['nombre'], True, (255, 255, 255))  # Renderiza el nombre del nivel

    # Dibuja el texto en la ventana
    ventana.blit(texto_nivel, (ANCHO_PANTALLA//2 - texto_nivel.get_width()//2, ALTO_PANTALLA//2 - 50))
    ventana.blit(nombre_nivel, (ANCHO_PANTALLA//2 - nombre_nivel.get_width()//2, ALTO_PANTALLA//2 + 20))

    pygame.display.flip()  # Actualiza la pantalla
    pygame.time.wait(2000)  # Espera 2 segundos

def renderizar_efectos_fondo(ventana, nivel):
    """Renderiza efectos especiales según el nivel."""
    if nivel == 4:  # Si es el nivel de lava
        for _ in range(5):  # Genera 5 efectos
            x = random.randint(0, ANCHO_PANTALLA)  # Posición horizontal aleatoria
            y = random.randint(Y_MIN + 10, ALTO_PANTALLA)  # Posición vertical aleatoria
            pygame.draw.circle(ventana, (255, 100, 0), (x, y), random.randint(2, 5))  # Dibuja un círculo

def main():
    """Función principal del juego."""
    pygame.init()  # Inicializa Pygame
    ventana = pygame.display.set_mode((ANCHO_PANTALLA, ALTO_PANTALLA))  # Crea la ventana del juego
    pygame.display.set_caption("Super Mario- Aventura por Niveles")  # Establece el título de la ventana

    # Intenta cargar las fuentes de texto
    try:
        fuente = pygame.font.Font(None, FUENTE_TAMANO_VIDAS)  # Fuente para las vidas
        fuente_game_over = pygame.font.Font(None, FUENTE_TAMANO_GAME_OVER)  # Fuente para Game Over
        fuente_nivel = pygame.font.Font(None, FUENTE_TAMANO_NIVEL)  # Fuente para el nivel
    except pygame.error:
        # Si hay un error, usa fuentes por defecto
        fuente = pygame.font.SysFont(None, FUENTE_TAMANO_VIDAS)
        fuente_game_over = pygame.font.SysFont(None, FUENTE_TAMANO_GAME_OVER)
        fuente_nivel = pygame.font.SysFont(None, FUENTE_TAMANO_NIVEL)

    # Inicializa el reloj y el mezclador de sonidos
    reloj = pygame.time.Clock()  # Crea un reloj para controlar la velocidad del juego
    pygame.mixer.init()  # Inicializa el mezclador de sonidos
    sonidos = cargar_sonidos()  # Carga los sonidos del juego
    jugador = Jugador()  # Crea un nuevo jugador
    todos_sprites, monedas, hongos, estrella, obstaculos, plataformas, goombas = reiniciar_nivel(jugador)  # Reinicia el nivel

    # Variables para el juego
    mensaje = ""  # Mensaje temporal para mostrar al jugador
    mensaje_tiempo = None  # Tiempo para mostrar el mensaje
    ultimo_goomba = datetime.now()  # Tiempo de la última aparición de un Goomba
    mostrar_transicion = True  # Indica si se debe mostrar la transición de nivel
    tiempo_transicion = datetime.now()  # Tiempo de la transición
    ejecutando = True  # Indica si el juego está en ejecución
    game_over = False  # Indica si el juego ha terminado

    # Bucle principal del juego
    while ejecutando:
        dt = reloj.tick(60) / 1000  # Controla la velocidad del juego
        tiempo_actual = datetime.now()  # Obtiene el tiempo actual
        config_nivel = NIVELES[jugador.nivel]  # Obtiene la configuración del nivel actual

        # Mostrar transición de nivel
        if mostrar_transicion and tiempo_actual - tiempo_transicion > timedelta(seconds=0.5):
            mostrar_transicion_nivel(ventana, jugador.nivel, fuente_nivel)  # Muestra la transición
            mostrar_transicion = False  # Desactiva la transición

        # Manejar eventos del juego
        for event in pygame.event.get():
            if event.type == pygame.QUIT:  # Si se cierra la ventana
                ejecutando = False  # Termina el juego
            elif event.type == pygame.KEYDOWN:  # Si se presiona una tecla
                if event.key == pygame.K_SPACE and not game_over:  # Si se presiona espacio y no está en Game Over
                    jugador.saltar()  # Hace que el jugador salte
                    sonidos['salto'].play()  # Reproduce el sonido de salto
                elif event.key == pygame.K_r and game_over:  # Si se presiona R y está en Game Over
                    # Reinicia el juego completo
                    jugador = Jugador()  # Crea un nuevo jugador
                    todos_sprites, monedas, hongos, estrella, obstaculos, plataformas, goombas = reiniciar_nivel(jugador)  # Reinicia el nivel
                    goombas.empty()  # Limpia el grupo de Goombas
                    game_over = False  # Restablece el estado de Game Over
                    mensaje = ""  # Limpia el mensaje
                    mostrar_transicion = True  # Activa la transición
                    tiempo_transicion = datetime.now()  # Reinicia el tiempo de transición

        if not game_over:  # Si el juego no ha terminado
            # Controles del jugador
            teclas = pygame.key.get_pressed()  # Obtiene el estado de las teclas
            if teclas[pygame.K_LEFT]:  # Si se presiona la tecla izquierda
                jugador.mover(-1)  # Mueve al jugador a la izquierda
            if teclas[pygame.K_RIGHT]:  # Si se presiona la tecla derecha
                jugador.mover(1)  # Mueve al jugador a la derecha

            # Actualiza las físicas del jugador
            jugador.actualizar_salto(config_nivel['gravedad_mod'], plataformas)  # Actualiza la posición del jugador

            # Generación de Goombas según el nivel
            intervalo = timedelta(seconds=config_nivel['intervalo_goomba'])  # Intervalo de aparición de Goombas
            if (len(goombas) < config_nivel['max_goombas'] and
                tiempo_actual - ultimo_goomba >= intervalo):  # Si se pueden generar más Goombas
                tipo_goomba = random.choice(config_nivel['tipos_goomba'])  # Elige un tipo de Goomba
                goomba = Goomba(tipo_goomba, config_nivel['velocidad_goomba'])  # Crea un nuevo Goomba
                goombas.add(goomba)  # Añade el Goomba al grupo
                todos_sprites.add(goomba)  # Añade el Goomba al grupo de todos los sprites
                ultimo_goomba = tiempo_actual  # Actualiza el tiempo de la última aparición
                enemigos_requeridos = 5 + jugador.nivel * 2  # Calcula el número de enemigos requeridos
                texto_objetivo = fuente.render(f'Objetivo: {jugador.goombas_derrotados}/{enemigos_requeridos}', True, config_nivel['texto'])  # Muestra el objetivo
                ventana.blit(texto_objetivo, (10, 210))  # Dibuja el texto en la ventana

            # Actualiza los sprites animados
            for moneda in monedas:
                if moneda.activo:  # Si la moneda está activa
                    moneda.update()  # Actualiza la animación de la moneda

            if estrella.activo:  # Si la estrella está activa
                estrella.update()  # Actualiza la animación de la estrella

            jugador.actualizar_inmunidad()  # Actualiza el estado de inmunidad del jugador
            goombas.update()  # Actualiza el estado de los Goombas

            # Colisiones con monedas
            monedas_recogidas = pygame.sprite.spritecollide(jugador, monedas, False)  # Verifica colisiones con monedas
            for moneda in monedas_recogidas:
                if moneda.activo:  # Si la moneda está activa
                    jugador.monedas_recolectadas += 1  # Incrementa el contador de monedas
                    jugador.puntos += 100  # Incrementa los puntos del jugador
                    moneda.activo = False  # Desactiva la moneda
                    moneda.rect.topleft = (-100, -100)  # Mueve la moneda fuera de la pantalla
                    sonidos['moneda'].play()  # Reproduce el sonido de recoger moneda
                    mensaje = f"Monedas: {jugador.monedas_recolectadas}"  # Actualiza el mensaje
                    mensaje_tiempo = datetime.now() + timedelta(seconds=2)  # Establece el tiempo para mostrar el mensaje

            # Colisiones con hongos
            hongos_recogidos = pygame.sprite.spritecollide(jugador, hongos, False)  # Verifica colisiones con hongos
            for hongo in hongos_recogidos:
                if hongo.activo:  # Si el hongo está activo
                    if hongo.tipo == 'crecimiento':  # Si es un hongo de crecimiento
                        jugador.crecer()  # Hace crecer al jugador
                        jugador.puntos += 200  # Incrementa los puntos del jugador
                        sonidos['powerup'].play()  # Reproduce el sonido de poder
                        mensaje = "¡Creciste!"  # Actualiza el mensaje para indicar que el jugador ha crecido
                    elif hongo.tipo == 'vida':  # Si es un hongo que otorga vida
                        jugador.vidas += 1  # Incrementa el número de vidas del jugador
                        jugador.puntos += 500  # Incrementa los puntos del jugador
                        sonidos['powerup'].play()  # Reproduce el sonido de poder
                        mensaje = "¡Vida extra!"  # Actualiza el mensaje para indicar que se ha ganado una vida
                    hongo.desactivar()  # Desactiva el hongo después de ser recogido
                    mensaje_tiempo = datetime.now() + timedelta(seconds=2)  # Establece el tiempo para mostrar el mensaje

            # Colisión con estrella
            if estrella.activo and jugador.rect.colliderect(estrella.rect):  # Si el jugador colisiona con la estrella
                jugador.activar_inmunidad()  # Activa la inmunidad del jugador
                jugador.puntos += 300  # Incrementa los puntos del jugador
                estrella.desactivar()  # Desactiva la estrella después de ser recogida
                sonidos['powerup'].play()  # Reproduce el sonido de poder
                mensaje = "¡Inmunidad activada!"  # Actualiza el mensaje para indicar que se ha activado la inmunidad
                mensaje_tiempo = datetime.now() + timedelta(seconds=2)  # Establece el tiempo para mostrar el mensaje

            # Reactivación de objetos
            for hongo in hongos:  # Revisa cada hongo en el grupo
                if (not hongo.activo and hongo.tiempo_reactivacion and
                    tiempo_actual > hongo.tiempo_reactivacion):  # Si el hongo no está activo y su tiempo de reactivación ha pasado
                    hongo.activar()  # Reactiva el hongo

            if (estrella.tiempo_reactivacion and
                tiempo_actual > estrella.tiempo_reactivacion):  # Si la estrella tiene un tiempo de reactivación
                estrella.resetear()  # Reinicia la posición de la estrella

            # Colisiones con Goombas
            if not jugador.inmune:  # Si el jugador no es inmune
                golpes = pygame.sprite.spritecollide(jugador, goombas, False)  # Verifica colisiones con Goombas
                for goomba in golpes:
                    if (jugador.velocidad_y > 0 and
                        jugador.rect.bottom <= goomba.rect.top + 10):  # Si el jugador salta sobre el Goomba
                        if goomba.golpear():  # Si el Goomba es derrotado
                            jugador.goombas_derrotados += 1  # Incrementa el contador de Goombas derrotados
                            jugador.puntos += 150 * jugador.nivel  # Incrementa los puntos del jugador
                            mensaje = f"¡Goomba derrotado! +{150 * jugador.nivel}"  # Actualiza el mensaje
                            mensaje_tiempo = datetime.now() + timedelta(seconds=2)  # Establece el tiempo para mostrar el mensaje
                        jugador.velocidad_y = FUERZA_SALTO // 2  # Ajusta la velocidad vertical del jugador
                    else:  # Si hay una colisión lateral
                        goomba.kill()  # Elimina el Goomba
                        sonidos['hit'].play()  # Reproduce el sonido de golpe
                        if jugador.tamano == 'grande':  # Si el jugador es grande
                            jugador.reducir()  # Reduce el tamaño del jugador
                            jugador.activar_inmunidad()  # Activa la inmunidad
                            mensaje = "¡Reducido a pequeño!"  # Actualiza el mensaje
                        else:  # Si el jugador es pequeño
                            jugador.vidas -= 1  # Resta una vida al jugador
                            if jugador.vidas <= 0:  # Si el jugador se queda sin vidas
                                game_over = True  # Cambia el estado a Game Over
                                sonidos['game_over'].play()  # Reproduce el sonido de Game Over
                            else:
                                jugador.rect.midbottom = (100, Y_MIN)  # Restaura la posición del jugador
                                jugador.activar_inmunidad()  # Activa la inmunidad
                        mensaje_tiempo = datetime.now() + timedelta(seconds=2)  # Establece el tiempo para mostrar el mensaje

            # Colisiones con obstáculos
            obstaculos_golpeados = pygame.sprite.spritecollide(jugador, obstaculos, False)  # Verifica colisiones con obstáculos
            if obstaculos_golpeados and not jugador.inmune:  # Si el jugador choca con un obstáculo y no es inmune
                if jugador.tamano == 'grande':  # Si el jugador es grande
                    jugador.reducir()  # Reduce el tamaño del jugador
                    jugador.activar_inmunidad()  # Activa la inmunidad
                    mensaje = "¡Chocaste con obstáculo!"  # Actualiza el mensaje
                else:  # Si el jugador es pequeño
                    jugador.vidas -= 1  # Resta una vida al jugador
                    if jugador.vidas <= 0:  # Si el jugador se queda sin vidas
                        game_over = True  # Cambia el estado a Game Over
                        sonidos['game_over'].play()  # Reproduce el sonido de Game Over
                    else:
                        jugador.rect.midbottom = (100, Y_MIN)  # Restaura la posición del jugador
                        jugador.activar_inmunidad()  # Activa la inmunidad
                mensaje_tiempo = datetime.now() + timedelta(seconds=2)  # Establece el tiempo para mostrar el mensaje

            # Lógica de progresión de nivel
            if jugador.goombas_derrotados >= 5 + jugador.nivel * 2:  # Si el jugador ha derrotado suficientes Goombas
                if jugador.subir_nivel():  # Intenta subir de nivel
                    sonidos['nivel_up'].play()  # Reproduce el sonido de subir de nivel
                    todos_sprites, monedas, hongos, estrella, obstaculos, plataformas, goombas = reiniciar_nivel(jugador)  # Reinicia el nivel
                    goombas.empty()  # Limpia el grupo de Goombas
                    mensaje = f"¡Nivel {jugador.nivel} desbloqueado!"  # Actualiza el mensaje
                    mensaje_tiempo = datetime.now() + timedelta(seconds=3)  # Establece el tiempo para mostrar el mensaje
                    mostrar_transicion = True  # Activa la transición
                    tiempo_transicion = datetime.now()  # Reinicia el tiempo de transición
                else:  # Si no se puede subir de nivel
                    game_over = True  # Cambia el estado a Game Over
                    mensaje = "¡JUEGO COMPLETADO!"  # Actualiza el mensaje de finalización del juego

        # Limpiar pantalla con el color de fondo del nivel
        ventana.fill(config_nivel['fondo'])  # Rellena la ventana con el color de fondo del nivel

        # Renderizar efectos de fondo especiales
        renderizar_efectos_fondo(ventana, jugador.nivel)  # Llama a la función para renderizar efectos

        # Dibujar el suelo
        pygame.draw.rect(ventana, config_nivel['suelo'], (0, Y_MIN, ANCHO_PANTALLA, ALTO_PANTALLA - Y_MIN))  # Dibuja el suelo

        # Dibujar todos los sprites en la pantalla
        for sprite in todos_sprites:
            if hasattr(sprite, 'activo') and not sprite.activo:  # Si el sprite no está activo
                continue  # Salta al siguiente sprite
            ventana.blit(sprite.image, sprite.rect)  # Dibuja el sprite en la ventana

        # Dibujar Goombas en la pantalla
        for goomba in goombas:
            ventana.blit(goomba.image, goomba.rect)  # Dibuja cada Goomba en la ventana

        # Información del jugador
        texto_vidas = fuente.render(f'Vidas: {jugador.vidas}', True, config_nivel['texto'])  # Muestra las vidas del jugador
        texto_puntos = fuente.render(f'Puntos: {jugador.puntos}', True, config_nivel['texto'])  # Muestra los puntos del jugador
        texto_nivel = fuente.render(f'Nivel: {jugador.nivel}', True, config_nivel['texto'])  # Muestra el nivel actual
        texto_monedas = fuente.render(f'Monedas: {jugador.monedas_recolectadas}', True, config_nivel['texto'])  # Muestra las monedas recolectadas
        texto_goombas = fuente.render(f'Goombas: {jugador.goombas_derrotados}', True, config_nivel['texto'])  # Muestra los Goombas derrotados

        # Dibuja la información del jugador en la ventana
        ventana.blit(texto_vidas, (10, 10))  # Dibuja las vidas
        ventana.blit(texto_puntos, (10, 50))  # Dibuja los puntos
        ventana.blit(texto_nivel, (10, 90))  # Dibuja el nivel
        ventana.blit(texto_monedas, (10, 130))  # Dibuja las monedas
        ventana.blit(texto_goombas, (10, 170))  # Dibuja los Goombas derrotados

        # Mostrar estado de inmunidad
        if jugador.inmune:  # Si el jugador es inmune
            tiempo_restante = (jugador.tiempo_inmunidad - datetime.now()).total_seconds()  # Calcula el tiempo restante
            if tiempo_restante > 0:  # Si aún queda tiempo de inmunidad
                texto_inmune = fuente.render(f'Inmunidad: {tiempo_restante:.1f}s', True, (255, 255, 0))  # Muestra el tiempo restante
                ventana.blit(texto_inmune, (10, 210))  # Dibuja el texto de inmunidad en la ventana

        # Mostrar mensajes temporales
        if mensaje_tiempo and datetime.now() < mensaje_tiempo:  # Si hay un mensaje que mostrar
            texto_mensaje = fuente.render(mensaje, True, (255, 255, 255))  # Renderiza el mensaje
            ventana.blit(texto_mensaje, (ANCHO_PANTALLA//2 - texto_mensaje.get_width()//2, 250))  # Dibuja el mensaje en la ventana

        # Pantalla de Game Over
        if game_over:  # Si el juego ha terminado
            overlay = pygame.Surface((ANCHO_PANTALLA, ALTO_PANTALLA))  # Crea una superficie para la pantalla de Game Over
            overlay.set_alpha(150)  # Establece la transparencia
            overlay.fill((0, 0, 0))  # Rellena la superficie con negro
            ventana.blit(overlay, (0, 0))  # Dibuja la superficie en la ventana

            if jugador.vidas <= 0:  # Si el jugador se queda sin vidas
                texto_game_over = fuente_game_over.render('GAME OVER', True, COLOR_GAME_OVER)  # Muestra el mensaje de Game Over
                texto_reiniciar = fuente.render('Presiona R para reiniciar', True, COLOR_TEXTO_BLANCO)  # Instrucciones para reiniciar
            else:  # Si el jugador ha ganado
                texto_game_over = fuente_game_over.render('¡VICTORIA!', True, (0, 255, 0))  # Muestra el mensaje de victoria
                texto_reiniciar = fuente.render('¡Completaste todos los niveles!', True, COLOR_TEXTO_BLANCO)  # Mensaje de finalización

            # Dibuja los mensajes de Game Over o victoria en la ventana
            ventana.blit(texto_game_over, (ANCHO_PANTALLA//2 - texto_game_over.get_width()//2, ALTO_PANTALLA//2 - 100))
            ventana.blit(texto_reiniciar, (ANCHO_PANTALLA//2 - texto_reiniciar.get_width()//2, ALTO_PANTALLA//2 + 50))

            # Mostrar estadísticas finales
            texto_puntos_final = fuente.render(f'Puntos Finales: {jugador.puntos}', True, COLOR_TEXTO_BLANCO)  # Muestra los puntos finales
            texto_nivel_final = fuente.render(f'Nivel Alcanzado: {jugador.nivel}', True, COLOR_TEXTO_BLANCO)  # Muestra el nivel alcanzado
            ventana.blit(texto_puntos_final, (ANCHO_PANTALLA//2 - texto_puntos_final.get_width()//2, ALTO_PANTALLA//2 + 100))  # Dibuja los puntos finales
            ventana.blit(texto_nivel_final, (ANCHO_PANTALLA//2 - texto_nivel_final.get_width()//2, ALTO_PANTALLA//2 + 140))  # Dibuja el nivel alcanzado

        pygame.display.flip()  # Actualiza la pantalla

    pygame.quit()  # Cierra Pygame al finalizar el juego

# Punto de entrada del programa
if __name__ == "__main__":
    main()  # Llama a la función principal para iniciar el juego
