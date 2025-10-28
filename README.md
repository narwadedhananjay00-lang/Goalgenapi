# Goalgenapifrom fastapi import FastAPI
from pydantic import BaseModel
from google import genai
import os

# --- 1. Key Setup: Replit Secrets se key lenge ---
GEMINI_API_KEY = os.environ.get("GEMINI_API_KEY")

if not GEMINI_API_KEY:
    raise ValueError("GEMINI_API_KEY is not set in Replit Secrets.")

gemini_client = genai.Client(api_key=GEMINI_API_KEY)
app = FastAPI()

class GoalInput(BaseModel):
    goal: str

# --- 2. Fusion Prompt Logic ---
def generate_fusion_prompt(user_goal: str):
    prompt = f"""
    You are 'GoalGen', an AI Goal Search Engine. Your task is to analyze the user's dream goal and create a detailed, actionable success roadmap. The plan must be motivational and supportive.

    Goal to Analyze: "{user_goal}"

    Your final output must be structured clearly with two main markdown headings:
    
    ## GEMINI INSIGHTS
    List 3 current trending growth areas and 3 essential skills needed for this goal.
    
    ## ACTION PLAN
    Create a motivational, 6-month step-by-step roadmap. Break the first 3 months into weekly, actionable tasks and the last 3 months into milestones.
    """
    return prompt

# --- 3. API Endpoint (Thunkable Yahi Call Karega) ---
@app.post("/generate_roadmap")
def create_roadmap(data: GoalInput):
    user_goal = data.goal
    full_prompt = generate_fusion_prompt(user_goal)
    
    try:
        response = gemini_client.models.generate_content(
            model='gemini-2.5-flash',
            contents=full_prompt
        )
        
        # Final result ko JSON format mein Thunkable ko wapas bhejna
        return {
            "status": "success",
            "roadmap": response.text,
            "goal": user_goal
        }
    
    except Exception as e:
        return {
            "status": "error",
            "message": f"An error occurred: {e}"
        }

@app.get("/")
def home():
    return {"message": "GoalGen API Engine is Running! Ready for Thunkable connection."}
