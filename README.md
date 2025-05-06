# RAG 토이프로젝트

## 프로젝트 개요

이 프로젝트는 프론트엔드 개발자로써 RAG(Retrieval Augmented Generation) 를 이해하는 용도로 작성한 간단한 테스트용 RAG 토이 프로젝트입니다. 단순한 LLM API 호출을 넘어서, 벡터 데이터베이스, 임베딩, 컨텍스트 검색 등 RAG 아키텍처의 핵심 요소들을 이해하고 구현하기 위해 개발되었습니다.

AI 기술을 활용한 애플리케이션 개발 역량 강화와, 특히 문서 기반 질의응답 시스템의 내부 작동 원리를 실제 코드로 구현하여 이해도를 높이는 것을 목표로 합니다.

## 주요 기능

### 📚 문서 관리 시스템
- **문서 업로드 및 처리**: PDF 및 텍스트 파일 지원
- **문서 청킹**: 효율적인 검색을 위한 문서 분할 처리
- **카테고리 관리**: 문서를 분류하여 검색 범위 최적화
- **문서 및 청크 CRUD**: 문서와 개별 청크에 대한 완전한 관리 기능

### 🔍 검색 엔진
- **벡터 유사도 검색**: 임베딩 기반의 의미적 검색 구현
- **카테고리 필터링**: 특정 주제에 관련된 문서만 검색
- **자동 카테고리 감지**: 질문 내용에 따른 관련 카테고리 자동 선택
- **검색 결과 미리보기**: 검색된 문서의 관련 섹션 표시

### 🤖 LLM 통합
- **멀티 모델 지원**: OpenAI와 Google Gemini 모델 지원
- **컨텍스트 기반 응답**: 검색된 문서 정보를 기반으로 정확한 답변 생성
- **참조 문서 표시**: 답변에 사용된 원본 문서 소스 제공
- **신뢰도 표시**: 검색 결과와의 유사도 점수 표시

### ⚡ 성능 최적화
- **임베딩 캐싱**: 반복 계산 방지를 위한 임베딩 벡터 캐싱
- **효율적인 청킹**: 의미 단위 기반의 문서 분할
- **섹션 인식**: 문서의 구조적 정보 활용
- **중복 제거**: 동일 문서의 중복 참조 방지

## 기술 스택

### 백엔드
- **프레임워크**: Streamlit (데이터 앱 구축)
- **데이터베이스**: Supabase (PostgreSQL + pgvector 확장)
- **AI 모델**:
  - OpenAI Embeddings (text-embedding-ada-002)
  - Google Gemini (gemini-2.0-flash)
- **파일 처리**: PyPDF2 (PDF 파싱)

### 인프라
- **벡터 저장소**: Supabase (pgvector)
- **API 통합**: OpenAI API, Google Generative AI API

## 시스템 아키텍처

```
                              ┌─────────────────┐
                              │   Streamlit     │
                              │   Web Interface │
                              └────────┬────────┘
                                       │
                                       │
     ┌──────────────────┬──────────────┴───────┬─────────────────┐
     │                  │                      │                 │
┌────▼─────┐      ┌─────▼────┐           ┌─────▼─────┐    ┌─────▼─────┐
│ 문서 업로드 │      │ 문서 관리  │           │ RAG 검색   │    │ 질의응답   │
└────┬─────┘      └─────┬────┘           └─────┬─────┘    └─────┬─────┘
     │                  │                      │                 │
     │                  │                      │                 │
┌────▼─────┐      ┌─────▼────┐           ┌─────▼─────┐    ┌─────▼─────┐
│ 문서 청킹  │      │ 벡터 저장소 │◄─────────►│ 임베딩 생성 │    │  LLM 모델  │
└────┬─────┘      │ (Supabase)│           │  (OpenAI) │    │(Gemini/OAI)│
     │            └───────────┘           └───────────┘    └───────────┘
     │
┌────▼─────┐
│임베딩 캐싱 │
└───────────┘
```

## 프로젝트 구조

```
RAG/
├── src/                       # 소스 코드
│   ├── Home.py               # 메인 애플리케이션 (Streamlit)
│   ├── qa.py                 # 질의응답 시스템 코어
│   ├── db.py                 # 벡터 데이터베이스 인터페이스
│   ├── DocumentLoader.py     # 문서 처리 및 청킹
│   ├── embedding_cache.py    # 임베딩 캐싱 시스템
│   ├── hybrid_search.py      # 하이브리드 검색 구현
│   ├── category_config.py    # 카테고리 설정 관리
│   └── pages/                # 추가 페이지
│       └── 1_DB_Management.py # 문서 관리 페이지
├── supabase/                 # Supabase 관련 설정
│   └── functions/           
├── requirements.txt          # 패키지 의존성
└── .env                      # 환경 변수 (API 키 등)
```

## RAG 핵심 컴포넌트 구현

### 1. 문서 처리 파이프라인

```python
# DocumentLoader.py에서 구현된 문서 처리 파이프라인
def process_document(file, file_name, title, category):
    # 1. 파일 형식에 따른 텍스트 추출
    text = extract_text_from_file(file, file_name)
    
    # 2. 문서를 의미 있는 청크로 분할
    chunks = chunk_document(text)
    
    # 3. 각 청크에 메타데이터 추가
    processed_chunks = [{"content": chunk} for chunk in chunks]
    
    # 4. 문서 메타데이터 구성
    metadata = {
        "title": title,
        "category": category,
        "original_filename": file_name
    }
    
    return processed_chunks, metadata
```

### 2. 벡터 검색 엔진

```python
# db.py에서 구현된 벡터 검색
def search_similar(self, query_embedding, limit=5, category=None):
    # 1. 기본 쿼리 구성
    query = self.supabase.rpc(
        "match_chunks",
        {"query_embedding": query_embedding, "match_threshold": 0.5, "match_count": 10}
    )
    
    # 2. 카테고리 필터 적용
    if category:
        query = query.eq("category", category)
    
    # 3. 검색 실행 및 결과 반환
    response = query.execute()
    
    return response.data
```

### 3. 질의응답 시스템

```python
# qa.py에서 구현된 RAG 기반 질의응답
def ask(self, question, category=None):
    # 1. 질문 임베딩 생성
    query_embedding = self._create_embedding(question)
    
    # 2. 유사한 청크 검색
    similar_chunks = self.vector_store.search_similar(query_embedding)
    
    # 3. 컨텍스트 구성
    context = "\n\n".join([chunk["content"] for chunk in similar_chunks])
    
    # 4. LLM을 사용하여 답변 생성
    answer = self._create_chat_completion(
        "주어진 컨텍스트를 기반으로 질문에 답변해주세요.",
        f"컨텍스트:\n{context}\n\n질문: {question}"
    )
    
    # 5. 참조 문서 정보 구성 및 반환
    references = self._prepare_references(similar_chunks)
    
    return {"answer": answer, "documents": references}
```

## RAG 아키텍처 특징

### 1. 임베딩 캐싱 최적화

API 호출 비용과 시간을 절약하기 위해 한 번 생성한 임베딩을 캐시하는 시스템을 구현했습니다. 동일한 텍스트에 대해 임베딩을 재계산하지 않고 캐시에서 가져옵니다.

```python
# embedding_cache.py
def get(self, text, model_name):
    """캐시에서 임베딩 조회"""
    text_hash = self._hash_text(text)
    return self.cache.get((text_hash, model_name))

def set(self, text, model_name, embedding):
    """임베딩을 캐시에 저장"""
    text_hash = self._hash_text(text)
    self.cache[(text_hash, model_name)] = embedding
```

### 2. 카테고리 자동 감지

질문 내용에 따라 관련된 카테고리를 자동으로 선택하여 검색 범위를 최적화하는 기능을 제공합니다.

### 3. 하이브리드 검색

단순한 벡터 검색 외에도 키워드 기반 검색과 벡터 검색을 조합한 하이브리드 검색 알고리즘을 구현했습니다.

## 환경 설정

### 필수 환경 변수

프로젝트 실행을 위해 `.env` 파일에 다음 환경 변수를 설정해야 합니다:

```
OPENAI_API_KEY=your-openai-api-key
GEMINI_API_KEY=your-gemini-api-key
SUPABASE_URL=your-supabase-url
SUPABASE_KEY=your-supabase-key
```

### Supabase 설정

Supabase에 벡터 검색을 위한 SQL 함수를 설정해야 합니다:

```sql
-- 벡터 유사도 검색 함수
create or replace function match_chunks(
  query_embedding vector(1536),
  match_threshold float,
  match_count int
)
returns table (
  id uuid,
  document_id uuid,
  content text,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    chunks.id,
    chunks.document_id,
    chunks.content,
    1 - (chunks.embedding <=> query_embedding) as similarity
  from chunks
  where 1 - (chunks.embedding <=> query_embedding) > match_threshold
  order by similarity desc
  limit match_count;
end;
$$;
```

## 설치 및 실행

```bash
# 저장소 클론
git clone https://github.com/gopizza/Rag.git
cd Rag

# 가상환경 생성 및 활성화
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install -r requirements.txt

# Streamlit 애플리케이션 실행
streamlit run src/Home.py
```

## 사용 방법

1. **문서 추가하기**:
   - "DB 관리" 페이지로 이동
   - "문서 업로드" 섹션에서 PDF 또는 텍스트 파일 업로드
   - 문서 제목과 카테고리 설정 후 업로드

2. **질문하기**:
   - 메인 화면에서 질문 입력
   - 필요시 특정 카테고리 선택
   - 답변과 참조 문서 확인

3. **문서 관리**:
   - "DB 관리" 페이지에서 기존 문서 조회
   - 문서 편집, 삭제 또는 개별 청크 관리

## 프런트엔드 개발자를 위한 RAG 학습 포인트

이 프로젝트를 통해 프론트엔드 개발자는 다음과 같은 RAG 핵심 개념을 학습할 수 있습니다:

1. **임베딩과 벡터 데이터베이스의 이해**:
   - 텍스트를 벡터로 변환하여 의미적 검색을 가능하게 하는 원리
   - 벡터 데이터베이스의 구조와 쿼리 방식

2. **효과적인 문서 분할 전략**:
   - 크기 기반 vs 의미 기반 청킹의 차이점
   - 청크 크기와 검색 정확도의 관계

3. **LLM 프롬프트 엔지니어링**:
   - 검색된 컨텍스트를 효과적으로 프롬프트에 통합하는 방법
   - 할루시네이션(환각)을 줄이기 위한 프롬프트 작성 기법

4. **검색 결과 평가 및 최적화**:
   - 검색 결과의 관련성 평가 방법
   - 검색 파라미터 튜닝을 통한 성능 개선

