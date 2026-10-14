# RAG 구현

한성대학교 학사공지 PDF를 검색해서 답하는 대화형 RAG 챗봇을 만든 7주차 실습이다. 오픈소스 AI·SW 융합 프로젝트 수업에서 진행했다.

LLM은 로컬에서 돌리는 Ollama의 gemma3:4b를 썼다. API 키나 비용이 필요 없다.

## 구성

```text
.
├─ 6w/data/                 공지 PDF 2개 (공지1.pdf, 공지2.pdf)
└─ 7w/
   ├─ step1_ollama.py       Ollama 연결 확인
   ├─ step2_retrieval.py    PDF 청킹, 임베딩, Chroma 검색
   ├─ step3_basic_rag.py    검색 결과로 답변 생성 (Basic RAG)
   ├─ step4_chat_rag.py     이전 대화를 반영하는 대화형 RAG
   ├─ app.py                Streamlit 채팅 화면
   ├─ requirements.txt
   ├─ 구현설명.md           단계별 구현과 값을 고른 이유
   └─ 실행및결과.md         단계별 실행 결과와 관찰
```

코드가 상위 폴더의 `6w/data`를 읽기 때문에 이 폴더 구조를 그대로 둬야 한다.

## 동작 방식

1. PDF를 페이지 단위로 읽는다.
2. 700자 조각(chunk)으로 자르고, 조각끼리 100자씩 겹친다.
3. 다국어 임베딩 모델 `paraphrase-multilingual-MiniLM-L12-v2`로 조각을 벡터로 바꿔 Chroma에 저장한다.
4. 질문도 벡터로 바꿔서 가장 가까운 조각 3개를 찾는다.
5. 찾은 조각과 질문을 프롬프트에 넣어 gemma3:4b에 보낸다. 대화형에서는 이전 대화도 함께 넣는다.
6. 프롬프트에서 문서에 없는 내용은 "제공된 문서에서 확인할 수 없습니다."라고 답하게 한다.

대화형(step4, app)은 후속 질문을 처리하려고 검색할 때 이전 질문과 현재 질문을 합친다.

## 실행 방법

환경은 Windows 11, Python 3.11, Ollama 0.40.4다. 수업 안내서는 Python 3.12지만 3.11에서 돌려 보았다.

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

처음 실행할 때 임베딩 모델을 내려받아서 시간이 더 걸린다. Chroma는 저장 경로 없이 메모리에서만 쓰기 때문에 실행할 때마다 임베딩을 새로 만든다.

## 결과 요약

자세한 출력은 [7w/실행및결과.md](7w/실행및결과.md)에 있다.

- step4에서 "그럼 제출 기한은?"이라는 후속 질문에 "2026년 10월 18일(일)까지"라고 맞게 답했다. 이전 질문을 검색어에 합친 덕분에 일정표 조각이 검색됐다.
- step3의 "인턴십 신청 기간은 언제인가요?"에는 틀리게 답했다. 다른 공지(IPP 일학습병행)의 마감일인 10월 7일을 말했다. 정답 조각이 상위 3개에 들지 못한 것이 원인이다. 자세한 분석은 실행및결과.md에 적었다.
- 검색이 정답 조각을 찾지 못한 경우에 지어내지 않고 "제공된 문서에서 확인할 수 없습니다."라고 답하는 사례도 있었다.
- 모델이 작아서(4B) 가끔 문장이 부정확하거나 답이 중간에 끊긴다. 이것도 실행및결과.md에 적었다.

## 출처

수업 저장소 [hjlee-crypto/2026-ai-sw-convergence](https://github.com/hjlee-crypto/2026-ai-sw-convergence)의 7주차 실습 자료를 따라 빈칸을 채워 구현했다. 데이터 PDF도 그 저장소의 `6w/data`에서 가져왔다.
