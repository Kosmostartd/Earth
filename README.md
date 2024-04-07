# Earth
import pygame as pg
import numpy as np
from math import cos, sin, pi


clock = pg.time.Clock()
FPS=10

WIDTH=800
HEIGHT=800

R=250
MAP_WIDTH=139
MAP_HEIGHT=34

pg.init()

my_font = pg.font.SysFont('arial', 14)

with open('earth_W140_H35.txt', 'r') as file:
    data = [file.read().replace('\n', '')]

ascii_chars = []
for line in data:
    for char in line:
        ascii_chars.append(char)

ascii_chars1=ascii_chars[::-1]

class Projection:
    def __init__(self, width, height):
        self.width=width
        self.height=height
        self.screen = pg.display.set_mode((width, height))
        self.background=(10,10,60)
        pg.display.set_caption('3D EARTH')
        self.surfaces={}
        self.text = ascii_chars1

    def addSurface(self,name,surface):
        self.surfaces[name]=surface

    def display(self):
        self.screen.fill(self.background)
        for surface in self.surfaces.values():
            i = 0
            for node in surface.nodes:

                self.text_surface = my_font.render(self.text[i], False, (0, 255, 0))
                if i > MAP_WIDTH - 1 and i < (MAP_WIDTH * MAP_HEIGHT - MAP_WIDTH) and node[1] > 0:
                    self.screen.blit(self.text_surface, (WIDTH // 2 + int(node[0]), HEIGHT // 2 + int(node[2])))
                i += 1

    def rotateAll(self,theta):
        for surface in self.surfaces.values():
            center = surface.findCenter()

            c=np.cos(theta)
            s=np.sin(theta)

            matrix= np.array([[c,-s,0,0],
                              [s,c,0,0],
                              [0,0,1,0],
                              [0,0,0,1]])
            surface.rotate(center,matrix)

class Object:
    def __init__(self):
        self.nodes=np.zeros((0,4))

    def addNodes(self, node_array):
        ones_column=np.ones((len(node_array),1))
        ones_added= np.hstack((node_array, ones_column))
        self.nodes=np.vstack((self.nodes, ones_added))

    def findCenter(self):
        mean =self.nodes.mean(axis=0)
        return mean

    def rotate(self,center, matrix):
        for i, node in enumerate(self.nodes):
            self.nodes[i]=center+np.matmul(matrix, node)

xyz=[]

for i in range(MAP_HEIGHT+1):
    lat = (pi/MAP_HEIGHT)*i
    for j in range(MAP_WIDTH+1):
        lon=(2*pi/MAP_WIDTH)*j
        x=round(1.2*R*sin(lat)*cos(lon),2)
        y=round(1.2*R*sin(lat)*sin(lon),2)
        z=round(R*cos(lat),2)
        xyz.append((x,y,z))


spin=0

running=True
while running:
    clock.tick(FPS)

    pv = Projection(WIDTH,HEIGHT)

    globe = Object()
    globe_nodes=[i for i in xyz]
    globe.addNodes(np.array(globe_nodes))
    pv.addSurface('globe',globe)
    pv.rotateAll(spin)
    pv.display()



    for event in pg.event.get():
        if event.type == pg.QUIT:
            running=False

    pg.display.update()
    spin+=0.05
