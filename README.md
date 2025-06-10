
#  OpenSW-ComfyUI 개선  
*오픈소스SW기초 6분반 9조*

---

##  프로젝트 소개

  본 프로젝트에서는 **Hugging Face 기반의 SentenceTransformer 및 KeyBERT 모델**을 활용한 자연어 파싱 로직을 **ComfyUI의 노드 시스템 내에 직접 통합**하였습니다. 사용자는 **한글로 입력한 문장을 DeepL API**를 통해 영어로 번역하고, 이후 문장을 파싱·후처리하여 **CLIPTextEncode 노드로 전달**되도록 처리 흐름을 구성하였습니다. 또한, **React와 Flask**를 기반으로 한 간결한 웹 기반 GUI를 구현하여, 복잡한 노드 구성을 숨기고 **이미지 크기, 시드값, CFG 등 핵심 설정만으로** 손쉽게 이미지 생성을 실행할 수 있도록 하였습니다.

---


<details>
<summary> 자연어 문장 내부 처리 흐름</summary>

--- 
본 프로젝트는 사용자가 입력한 한글 문장을 자동으로 번역하고, 파싱 및 후처리를 거쳐 CLIP 기반 텍스트 인코딩으로 연결한 후, 최종적으로 이미지를 생성하는 전체 파이프라인을 구성합니다. 각 단계는 다음과 같이 구성되어 있습니다:

1. **언어 감지 및 번역**  
- langdetect로 입력 언장의 언어를 판별  
- 한글인 경우 DeepL API를 사용해 자연스러운 영어 문장으로 자동 번역

2. **구문 파싱 및 키워드 추출**  
- KeyBERT + SentenceTransformer로 의미 있는 구문 후보 추출  
- cosine similarity 기반 중복 제거  
- Spacy + Matcher를 활용해 명사구, 인물 정보, 동명사 등을 추가 삽입

3. **구문 병합 및 강조 처리**  
- 연관된 구문 병합  
- `:1.3`, `:1.5` 형식으로 중요 구문 가중치 강조

4. **CLIP 텍스트 인코딩**  
- 키워드 시퀀스를 CLIPTextEncode 노드로 전달  
- `tokenize()` 및 `encode_from_tokens_scheduled()` 수행 → CONDITIONING 생성

5. **이미지 생성**  
- CONDITIONING을 기반으로 KSampler → VAEDecode를 통해 이미지 생성  
- 필요시 영역 설정/결합 등 조건 제어 가능

</details>

---

<details>
<summary> GUI 내부 처리 흐름</summary>


---
본 GUI는 React + Flask 기반으로 작동하며, 사용자의 입력을 받아 텍스트 처리부터 이미지 생성까지 자동화된 워크플로우를 구성합니다.

1. **사용자 입력 (React UI)**  
- 프롬프트 문장 + 이미지 설정값 입력  
- `POST /generate`로 Flask에 요청

2. **백엔드 처리 (Flask)**  
- 입력 JSON을 ComfyUI의 `/prompt` API로 전달  
- Flask는 중계자 역할만 수행 (자연어 처리 X)

3. **이미지 생성 (ComfyUI 커스텀 노드)**  
- DeepL 번역 → Hugging Face 파싱 → Spacy 후처리 → CLIP 인코딩  
- KSampler + VAEDecode로 최종 이미지 생성

4. **응답 반환 및 출력**  
- 생성 이미지 경로 or base64를 React에 반환  
- React UI에서 이미지 표시

</details>

---

##  기술적 장점
- **DeepL, Hugging Face, Spacy, KeyBERT**를 결합한 파이프라인으로 정교한 프롬프트 파싱이 가능  
- **번역, 키워드 추출, 구문 병합 등 전처리 자동화** → CLIPTextEncode에 최적화된 입력 생성  
- 불필요한 키워드 제거 및 **중요 구문 강조(가중치 부여)** → 이미지 품질 향상

---

##  사용자 편의성
- **한글 문장만 입력**하면 모든 처리가 자동으로 이루어져 사용이 매우 간편  
- **React + Flask 기반 GUI**를 통해 이미지 크기, 시드값 등 최소한의 옵션만 입력  
- 복잡한 ComfyUI 노드를 몰라도 **누구나 쉽게 이미지 생성 가능**

---

##  통합 GUI 아키텍처

<img width="822" alt="스크린샷 2025-06-10 오전 12 23 48" src="https://github.com/user-attachments/assets/67055d50-f638-434b-b0e8-7d60c4d462e0" />


---

##  구동 영상

[![프로젝트 시연 영상](https://img.youtube.com/vi/jIUUPcVcwEo/0.jpg)](https://www.youtube.com/embed/jIUUPcVcwEo?si=ldfF0CRASY1bH_cQ)

<!-- 또는 HTML iframe 사용 시
<iframe width="560" height="315" src="https://www.youtube.com/embed/jIUUPcVcwEo?si=ldfF0CRASY1bH_cQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
-->

---

##  실행 방법

아래 순서대로 백엔드 서버와 프론트엔드를 실행하면 웹 인터페이스를 통해 이미지를 생성할 수 있습니다.

### 1. ComfyUI 실행
```bash
# ComfyUI 루트 디렉토리에서
python main.py
```

### 2. Flask 백엔드 실행
```bash
# 백엔드 디렉토리에서 (예: backend/)
python app.py
```

### 3. 프론트엔드(React) 실행
```bash
# 프론트엔드 디렉토리에서 (예: frontend/)
npm install
npm run start
```

###  접속 방법
- 브라우저에서 `http://localhost:3000` 접속  
- 프롬프트 입력 후 이미지 자동 생성

---

## 개발 기간

2025년 5월 ~ 6월 (약 2개월)

---

##  팀원 및 역할

###  정유찬 (팀장)  
- ComfyUI 프롬프트에 적합한 다양한 NLP 모델(Hugging Face 기반)의 파싱 성능 실험  
- 비교 기준 수립, 실험 결과 문서화  
- 통합 GUI 설계 및 구현  
- 1차, 2차 발표 및 PPT 제작  
- [GitHub 프로필](https://github.com/uchanni/OpenSW-ComfyUI-)

---

###  김민준 (팀원)  
- Hugging Face 모델 및 DeepL API를 ComfyUI 노드에 연동하는 초기 구조 구현  
- KeyBERT 및 Spacy 기반 후처리 로직 설계 및 구현  
- 최종 키워드 품질 향상 기여  
- 3차 발표 및 PPT 제작  
- [GitHub 프로필](https://github.com/mjkim1128/OpenSW-ComfyUI-)

---

##  부록

###  성능개선 테스트 시트 (OneDrive)  
[👉 테스트 시트 바로가기](https://o365sen-my.sharepoint.com/:x:/g/personal/lamborghiner_o365sen_net/EYObqDJVUp9LtYEW4XkPoPABKSqXfGhByycejAsxFGGPVw?e=MYqfCQ)
