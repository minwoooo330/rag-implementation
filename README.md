# RAG 구현

한성대 학사공지 PDF 2개로 만든 대화형 RAG 챗봇이다. 오픈소스 AI·SW 융합 프로젝트 수업 7주차 실습이고, LLM은 Ollama로 로컬에서 돌리는 gemma3:4b를 썼다. API 키는 필요 없다.

![앱 실행 화면](7w/images/app.png)

## 폴더

```text
.
├─ 6w/data/                 공지 PDF 2개
├─ 7w/
│  ├─ step1_ollama.py       Ollama 연결 확인
│  ├─ step2_retrieval.py    PDF 자르기, 임베딩, Chroma 검색
│  ├─ step3_basic_rag.py    검색 결과로 답 만들기
│  ├─ step4_chat_rag.py     이전 대화를 반영하는 대화형 RAG
│  ├─ app.py                Streamlit 채팅 화면
│  ├─ requirements.txt
│  ├─ images/app.png
│  └─ 구현설명.md
└─ 실행및결과.md
```

코드가 상위 폴더의 `6w/data`를 읽기 때문에 이 구조를 바꾸면 안 된다.

## 어떻게 돌아가나

PDF를 700자 정도로 잘라서 조각을 만들고, 조각마다 임베딩 모델로 숫자 목록(벡터)을 만들어 Chroma에 넣는다. 질문이 들어오면 질문도 벡터로 바꿔서 가장 가까운 조각 3개를 찾고, 그 조각과 질문을 gemma3:4b에 같이 보내 답을 받는다. step4와 앱은 후속 질문을 처리하려고 이전 질문과 지금 질문을 붙여서 검색한다.

자세한 설명은 [7w/구현설명.md](7w/구현설명.md)에 적었다.

## 실행

환경은 Windows 11, Python 3.11, Ollama 0.40.4다.

```powershell
ollama pull gemma3:4b

cd 7w
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt

python step1_ollama.py
python step2_retrieval.py
python step3_basic_rag.py
python step4_chat_rag.py
streamlit run app.py
```

처음 실행할 때는 임베딩 모델을 내려받느라 오래 걸린다. Chroma를 메모리에서만 쓰기 때문에 실행할 때마다 임베딩을 새로 만든다.

## 결과 메모

출력은 [실행및결과.md](실행및결과.md)에 있다. 몇 가지만 적으면 이렇다.

- step4에서 "그럼 제출 기한은?"이라고 물으면 10월 18일이라고 맞게 답한다.
- step3은 인턴십 신청 기간을 물었더니 다른 공지(IPP 일학습병행)의 마감일인 10월 7일을 답했다. 앱 캡처에서도 같은 날짜가 한 번 나왔다. 검색이 정답 조각을 가져오지 못한 것이 원인으로 보인다.
- 4B 모델이라 답이 중간에 끊기거나 표현이 어색할 때가 있다.

수업 저장소 [hjlee-crypto/2026-ai-sw-convergence](https://github.com/hjlee-crypto/2026-ai-sw-convergence)의 7주차 자료를 따라 빈칸을 채웠고, PDF도 그 저장소의 `6w/data`에서 가져왔다.
