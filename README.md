# 🔦 Guard Blackout

# 🌎 Global Solution 2025 - PHYSICAL COMPUTING IOT

## 👥 Integrantes

- **Felipe Capriotti** – RM98460  
- **Manoella Herrerias Waideman** – RM98906  
- **Victor Hugo Andrade** – RM550996  

---

## 📌 Descrição do Projeto

**Guard Blackout** é uma plataforma integrada para detecção, relato e visualização de apagões em tempo real, combinando tecnologias de visão computacional, sensores IoT e geolocalização.

---

## 🚨 1. Problema & Oportunidade

Quedas de energia são frequentes no Brasil, especialmente em épocas de tempestade. Nessas situações críticas, a comunicação falha – o que compromete o socorro e a resposta rápida.

Entretanto:

- A **API de Dados Abertos da ANEEL** já fornece dados de interrupções em tempo real.
- Há **mais de 1 milhão de smart-meters** somente no Paraná.
  
Esses fatores abrem espaço para uma **solução crowd-sourced, inteligente e acessível**.

---

## 💡 2. Proposta de Valor

**Guard Blackout** funciona como um “Waze para blecautes”, com cinco camadas:

1. **📱 Mobile App (React Native)**  
   - Relato de apagões com um toque  
   - Funciona offline  
   - Reconhece o gesto "SOS" (braço em Y) via MediaPipe  

2. **📡 IoT / BLE Beacon**  
   - Alerta de luminosidade  
   - Detecção do gesto "SOS" para sinalizar falhas elétricas  

3. **🧠 Backend (.NET Microservices)**  
   - Agrupamento de relatos usando DBSCAN  
   - Geração de zonas de blackout em tempo real  
   - API pública REST e GraphQL  

4. **🖥 Painel para Concessionárias (WPF/Blazor)**  
   - Visualização com mapa de calor  
   - Indicadores como MTTR e SAIDI  

5. **🌐 Metaverso (Unreal Engine)**  
   - Vídeo imersivo (~30s) simulando um blackout com gesto "SOS"  
   - Iluminação de emergência automática  

---

## 🎥 Vídeo Demonstrativo

📺 Assista aqui: [Gauard Blackout – Demonstração](https://drive.google.com/file/d/1GawSiY3aAQwXm53AgO14qamITB1No1sY/view?usp=drive_link)

---

## 🧠 Tecnologias Utilizadas

- Python + OpenCV + MediaPipe
- C# (.NET Microservices e Blazor/WPF)
- React Native
- Unreal Engine
- BLE / IoT
- API da ANEEL (dados abertos)

---

## 🧪 Demonstrações de Código para atividade de IOT
### 🔆 Código 1: Alerta de Luminosidade

```python
import cv2
import numpy as np

def calcular_brilho(frame):
    # Converte para escala de cinza
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    # Retorna a média de brilho
    return np.mean(gray)

# Abrir a câmera
cap = cv2.VideoCapture(0)

if not cap.isOpened():
    print("Erro ao acessar a câmera")
    exit()

print("Monitorando brilho... Pressione 'q' para sair")

while True:
    ret, frame = cap.read()
    if not ret:
        print("Erro ao capturar o frame")
        break

    brilho = calcular_brilho(frame)
    print(f"Brilho atual: {brilho:.2f}")

    # Mostra o frame
    cv2.imshow('Camera', frame)

    # Verifica se a luz apagou (limite ajustável, ex: 10)
    if brilho < 15:
        from win10toast import ToastNotifier
        toaster = ToastNotifier()
        toaster.show_toast("Alerta", "Luz apagada detectada!", duration=5)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```
### 🆘 Código 2 : Alerta SOS (gesto em Y)

```python
import cv2
import mediapipe as mp
import numpy as np
from win10toast import ToastNotifier

# Função para calcular brilho do frame
def calcular_brilho(frame):
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    return np.mean(gray)

# Inicializa MediaPipe Pose
mp_drawing = mp.solutions.drawing_utils
mp_pose = mp.solutions.pose

toaster = ToastNotifier()
alert_shown_brilho = False  # Para alerta brilho
alert_shown_gesto = False   # Para alerta gesto

def detectar_braço_em_Y(landmarks, image_width, image_height):
    right_shoulder = landmarks[mp_pose.PoseLandmark.RIGHT_SHOULDER]
    right_elbow = landmarks[mp_pose.PoseLandmark.RIGHT_ELBOW]
    right_wrist = landmarks[mp_pose.PoseLandmark.RIGHT_WRIST]

    left_shoulder = landmarks[mp_pose.PoseLandmark.LEFT_SHOULDER]
    left_elbow = landmarks[mp_pose.PoseLandmark.LEFT_ELBOW]
    left_wrist = landmarks[mp_pose.PoseLandmark.LEFT_WRIST]

    rs_y = right_shoulder.y * image_height
    re_y = right_elbow.y * image_height
    rw_y = right_wrist.y * image_height

    ls_y = left_shoulder.y * image_height
    le_y = left_elbow.y * image_height
    lw_y = left_wrist.y * image_height

    right_arm_up = rw_y < rs_y and re_y < rs_y
    left_arm_up = lw_y < ls_y and le_y < ls_y

    return right_arm_up and left_arm_up

cap = cv2.VideoCapture(0)
if not cap.isOpened():
    print("Erro ao acessar a câmera")
    exit()

with mp_pose.Pose(min_detection_confidence=0.5, min_tracking_confidence=0.5) as pose:
    print("Monitorando brilho e gesto SOS - Pressione 'q' para sair.")
    while True:
        ret, frame = cap.read()
        if not ret:
            print("Erro ao capturar o frame")
            break

        # Cálculo do brilho
        brilho = calcular_brilho(frame)
        print(f"Brilho atual: {brilho:.2f}")

        # Converte para RGB para MediaPipe
        image_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        image_rgb.flags.writeable = False

        results = pose.process(image_rgb)

        image_rgb.flags.writeable = True
        image = cv2.cvtColor(image_rgb, cv2.COLOR_RGB2BGR)

        # Detecta gesto
        if results.pose_landmarks:
            mp_drawing.draw_landmarks(image, results.pose_landmarks, mp_pose.POSE_CONNECTIONS)

            if detectar_braço_em_Y(results.pose_landmarks.landmark, image.shape[1], image.shape[0]):
                cv2.putText(image, 'Gesto SOS detectado!', (10, 30),
                            cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2, cv2.LINE_AA)
                if not alert_shown_gesto:
                    toaster.show_toast("Alerta", "Gesto SOS detectado!", duration=5)
                    alert_shown_gesto = True
            else:
                alert_shown_gesto = False
```
        # Alerta de brilho baixo
        if brilho < 15:
            if not alert_shown_brilho:
                toaster.show_toast("Alerta", "Luz apagada detectada!", duration=5)
                alert_shown_brilho = True
```
