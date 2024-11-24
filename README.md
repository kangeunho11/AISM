import streamlit as st
import openai
from dotenv import load_dotenv
import os
import re

# 환경 변수 로드
load_dotenv()

# OpenAI API 키 설정
openai.api_key = os.getenv("OPENAI_API_KEY")
if not openai.api_key:
    st.error("API 키가 설정되지 않았습니다. .env 파일을 확인하세요.")

def remove_duplicates(text):
    """중복된 단어나 문장을 제거"""
    words = text.split()
    seen = set()
    result = []
    for word in words:
        if word not in seen:
            result.append(word)
            seen.add(word)
    return " ".join(result)

def post_process_translation(text):
    """번역된 텍스트를 자연스럽게 후처리하고 줄 바꿈 적용"""
    # 불필요한 공백 제거
    text = re.sub(r'\s+', ' ', text).strip()

    # 예시: 어색한 표현을 더 자연스럽게 바꿔주는 코드
    text = text.replace("칵테일 레시피", "칵테일 만드는 법").replace("추천", "제안")
    
    # 중복 단어 제거
    text = remove_duplicates(text)
    
    # 줄 바꿈 추가: 각 문장 끝에 '\n'을 추가하여 가독성 향상
    text = text.replace(". ", ".\n")  # 문장 끝에 '.' 다음에 줄 바꿈 추가
    text = text.replace("주의", "\n주의")  # '주의' 앞에 줄 바꿈 추가

    return text

# OpenAI를 활용한 함수들
def generate_cocktail_recipe(ingredients):
    """재료를 기반으로 칵테일 레시피 생성"""
    prompt = f"{ingredients}를(을) 사용한 칵테일 레시피를 한국어로 만들어주세요."
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7
        )
        recipe = response.choices[0].message['content'].strip()
        return post_process_translation(recipe)  # 한국어로 응답 후 후처리
    except Exception as e:
        return f"Error: {e}"

def recommend_pairing(food):
    """음식에 어울리는 술 추천"""
    prompt = f"{food}에 어울리는 술을 한국어로 추천해주세요."
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.7
        )
        pairing = response.choices[0].message['content'].strip()
        return post_process_translation(pairing)  # 한국어로 응답 후 후처리
    except Exception as e:
        return f"Error: {e}"

# Streamlit 앱 UI
st.title("AI 소믈리에")
tab1, tab2 = st.tabs(["🍸 칵테일 생성", "🍴 음식 페어링"])

# 탭 1: 맞춤형 칵테일 레시피 생성
with tab1:
    st.header("맞춤형 칵테일 생성")
    ingredients = st.text_input("사용 가능한 재료를 입력하세요:")
    if st.button("레시피 생성"):
        if ingredients:
            recipe = generate_cocktail_recipe(ingredients)
            st.write("추천 레시피: ")
            st.text_area("번역된 레시피", recipe, height=300)
        else:
            st.warning("재료를 입력해주세요.")

# 탭 2: 음식과 술 페어링 추천
with tab2:
    st.header("음식과 술 페어링")
    food = st.text_input("음식을 입력하세요:")
    if st.button("페어링 추천"):
        if food:
            pairing = recommend_pairing(food)
            st.write("추천 페어링: ")
            st.text_area("번역된 페어링", pairing, height=300)
        else:
            st.warning("음식을 입력해주세요.")
