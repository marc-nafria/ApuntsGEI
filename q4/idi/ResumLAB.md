# Resum examen LAB

#### Fragment shader:

```cpp
void main()
{      
    vec3 color = Ambient();
    vec3 L = -(vertexSCO.xyz);
    L = normalize(L);
    vec3 NnormalSCO = normalize(normalSCO);

    color += Difus(NnormalSCO, L, colFocus);
    color += Especular(NnormalSCO, L, vertexSCO, colFocus);
    FragColor = vec4(color, 1.0);
}
```

**A recordar:**

+ El vector **L** es calcula com `posFocusSCO - vertexSCO`.

+ Si el focus de llum està en la camara, llavors `posFocusSCO = (0, 0, 0)` en coordenades de l'observador.

+ El vector normal del fragment **s'ha de normalitzar**: `vec3 NnormalSCO = normalize(normalSCO)`.

---

#### Vertex shader

```cpp
mat3 normalMatrix = inverse(transpose(mat3 (view * TG)));
    normalSCO = vec3(normalMatrix * normal);
    normalSCO = normalize(normalSCO);
    vertexSCO = view * TG * vec4(vertex, 1);
    gl_Position = proj * view * TG * vec4 (vertex, 1.0);
```

**A recordar**

+ EL vector normal del vertex s'ha de pasar a **SCO**, mitjançant la **normalMatrix**: `nomalMatrix = inverse( transpose( mat3( view * TG ) ) )`.

+ El vertexSCO es calcula de la seguent manera: `vertexSCO = view * TG * vec4(vertex, 1)`.

---

#### Model Transform

```cpp
glm::mat4 TG = glm::mat4(1.0)
TG = rotate(TG, angle, glm::vec3(0, 1.0, 0));
TG = translate (TG, glm::vec3(1.0, 5.0, 0.0));
TG = scale (TG, glm::vec3(escala_x, escala_y, escala_z));
glUniformMatrix4fv ( getUniformLocation(program->programId(), "TG") , 1, GL_FALSE, &TG[0][0]);
```

---

#### View Matrix

```cpp
glm::mat4 View;
glm::vec3 OBS = glm::vec3(10.0, 5.0, 0.0);
glm::vec3 VRP = glm::vec3(0.0, 5.0, 0.0);
glm::vec3 UP = glm::vec3(0.0, 1.0, 0.0);


View = lookAt(OBS, VRP, UP);
glUniformMatrix4fv (getUniformLocation(program-> progam.Id(), "View"), 1, GL_FALSE, &View[0][0]);
```

---

#### Project Matrix

```cpp
glm::mat4 Proj;

// perpectiva
Proj = perspective (FOV, ra, zNear, ZFar);

// otogonal
Proj = ortho (left, right, bottom, top, zNear, ZFar);

glUniformMatrix4fv(getUniformLocation(program->program.Id(), "Proj"), 1, GL_FALSE, &Proj[0][0]);
```

---

#### Pas de uniform (glm::vec3)

```cpp
glm::vec3 exemple = glm::vec3(1.0, 5.0, 3.0);

glUniform3f (gelUniformLocation(program->programId(), "exemple"), exemple.x, exemple.y, exemple.z);
```

---

#### Definir senyals i slots en un Widget

```cpp
class MyGLWidget:: public WidgetPare {
    Q_OBJECT

    public slots:
        void slotExemple();
        void slotExempleParametres (int parametre);

    signals:
        void signalExemple();
        void signalExempleParametres (int parametre);
}
```

> **Important:** Per asociar signals a slots del diferents widgets del nostre Form, ho hem de fer desde el **designer (QT)**. El designer també ens permetra afegir altres widgets al examen, respectant les regles de diseny estudiades.

---

#### Herencia de funcions (MyGLWidget.h, MyGLWidget.cpp)

Normalment tenim una classe widget pare i una filla. **Només poder editar la filla**

Per fer-ho heredem funcions. **Només podem heredar aquelles declarades `virtual` en el pare!**

# 
