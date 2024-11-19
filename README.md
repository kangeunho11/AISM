import streamlit as st
import openai
from dotenv import load_dotenv
import os

# 환경 변수 로드
load_dotenv()

# OpenAI API 키 설정
openai.api_key = os.getenv("OPENAI_API_KEY")

# Streamlit 앱 UI
st.title("AI 소믈리에")
tab1, tab2, tab3 = st.tabs(["칵테일 생성", "음식 페어링", "술 리뷰"])

# 탭 1: 맞춤형 칵테일 레시피 생성
with tab1:
    st.header("맞춤형 칵테일 생성")
    ingredients = st.text_input("사용 가능한 재료를 입력하세요:")
    if st.button("레시피 생성"):
        if ingredients:
            recipe = generate_cocktail_recipe(ingredients)
            st.write("추천 레시피: ", recipe)
        else:
            st.warning("재료를 입력해주세요.")

# 탭 2: 음식과 술 페어링 추천
with tab2:
    st.header("음식과 술 페어링")
    food = st.text_input("음식을 입력하세요:")
    if st.button("페어링 추천"):
        if food:
            pairing = recommend_pairing(food)
            st.write("추천 페어링: ", pairing)
        else:
            st.warning("음식을 입력해주세요.")

# 탭 3: 술 리뷰 및 추천
with tab3:
    st.header("술 리뷰 및 추천")
    alcohol_name = st.text_input("술 이름을 입력하세요:")
    if st.button("리뷰 보기"):
        if alcohol_name:
            review = get_alcohol_review(alcohol_name)
            st.write("술 리뷰: ", review)
        else:
            st.warning("술 이름을 입력해주세요.")

# OpenAI를 활용한 함수들
def generate_cocktail_recipe(ingredients):
    prompt = f"Create a cocktail recipe using the following ingredients: {ingredients}."
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response['choices'][0]['message']['content']

def recommend_pairing(food):
    prompt = f"Recommend alcohol pairings for the food: {food}."
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response['choices'][0]['message']['content']

def get_alcohol_review(alcohol_name):
    prompt = f"Summarize reviews and information about {alcohol_name}."
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": prompt}]
    )
    return response['choices'][0]['message']['content']
