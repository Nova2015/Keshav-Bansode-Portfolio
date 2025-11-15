"""
Smart Meal Planner & Grocery Automation Agent
Single-file example implementation for Kaggle Agents Intensive capstone.


Notes:
- This is a clear, runnable skeleton. Replace LLM/tool placeholders with real integrations (OpenAI, Google Search, Shopify API, etc.).
- Designed for clarity and demonstration: multi-agent orchestration, memory, tools, observability.
"""


import json
import os
import time
from typing import List, Dict, Any


# -----------------------------
# Config / User profile
# -----------------------------
USER_PROFILE = {
"name": "Keshav Bansode",
"diet": "vegetarian",
"cuisine_preferences": ["Indian", "Mediterranean"],
"allergies": [],
"servings_per_meal": 2,
"weekly_budget": 1500 # currency units
}


DATA_DIR = "./data"
os.makedirs(DATA_DIR, exist_ok=True)
# -----------------------------
# Simple Memory Bank (file-backed)
# -----------------------------
class MemoryBank:
	def __init__(self, path: str):
		self.path = path
		self._data = {"pantry": {}, "applied_plans": []}
		self._load()


	def _load(self):
		if os.path.exists(self.path):
			try:
				with open(self.path, "r") as f:
					self._data = json.load(f)
			except Exception:
				# start fresh if file is malformed
				self._data = {"pantry": {}, "applied_plans": []}


	def save(self):
		with open(self.path, "w") as f:
			json.dump(self._data, f, indent=2)


	def get_pantry(self) -> Dict[str, float]:
		return self._data.get("pantry", {})


	def update_pantry(self, item: str, qty: float):
		p = self._data.setdefault("pantry", {})
		p[item] = p.get(item, 0) + qty
		self.save()


	def set_pantry(self, pantry: Dict[str, float]):
		self._data["pantry"] = pantry
		self.save()


	def append_plan(self, plan: Dict[str, Any]):
		self._data.setdefault("applied_plans", []).append(plan)
		self.save()


memory = MemoryBank(os.path.join(DATA_DIR, "memory.json"))
# -----------------------------
# Observability (basic)
# -----------------------------
def log(msg: str):
	ts = time.strftime("%Y-%m-%d %H:%M:%S")
	print(f"[{ts}] {msg}")


# -----------------------------
# Tool Adapters (placeholders)
# Replace these with real API calls when integrating.
"""
Smart Meal Planner & Grocery Automation Agent
Single-file example implementation for Kaggle Agents Intensive capstone.

Notes:
- This is a clear, runnable skeleton. Replace LLM/tool placeholders with real integrations (OpenAI, Google Search, Shopify API, etc.).
- Designed for clarity and demonstration: multi-agent orchestration, memory, tools, observability.
"""

import json
import os
import time
from typing import List, Dict, Any


# -----------------------------
# Config / User profile
# -----------------------------
USER_PROFILE = {
	"name": "Keshav Bansode",
	"diet": "vegetarian",
	"cuisine_preferences": ["Indian", "Mediterranean"],
	"allergies": [],
	"servings_per_meal": 2,
	"weekly_budget": 1500,  # currency units
}


DATA_DIR = "./data"
os.makedirs(DATA_DIR, exist_ok=True)


# -----------------------------
# Simple Memory Bank (file-backed)
# -----------------------------
class MemoryBank:
	def __init__(self, path: str):
		self.path = path
		self._data = {"pantry": {}, "applied_plans": []}
		self._load()

	def _load(self):
		if os.path.exists(self.path):
			try:
				with open(self.path, "r") as f:
					self._data = json.load(f)
			except Exception:
				# start fresh if file is malformed
				self._data = {"pantry": {}, "applied_plans": []}

	def save(self):
		with open(self.path, "w") as f:
			json.dump(self._data, f, indent=2)

	def get_pantry(self) -> Dict[str, float]:
		return self._data.get("pantry", {})

	def update_pantry(self, item: str, qty: float):
		p = self._data.setdefault("pantry", {})
		p[item] = p.get(item, 0) + qty
		self.save()

	def set_pantry(self, pantry: Dict[str, float]):
		self._data["pantry"] = pantry
		self.save()

	def append_plan(self, plan: Dict[str, Any]):
		self._data.setdefault("applied_plans", []).append(plan)
		self.save()


memory = MemoryBank(os.path.join(DATA_DIR, "memory.json"))


# -----------------------------
# Observability (basic)
# -----------------------------
def log(msg: str):
	ts = time.strftime("%Y-%m-%d %H:%M:%S")
	print(f"[{ts}] {msg}")


# -----------------------------
# Tool Adapters (placeholders)
# Replace these with real API calls when integrating.
# -----------------------------


def llm_generate(prompt: str, temperature=0.2) -> str:
	"""Placeholder LLM call. Replace with OpenAI/Anthropic/etc."""
	log("LLM called")
	# Simple heuristic dummy response for demo purposes
	if "plan meals" in prompt.lower():
		return json.dumps({
			"Monday": {"breakfast": "Oats with fruit", "lunch": "Vegetable pulao", "dinner": "Paneer curry"},
			"Tuesday": {"breakfast": "Poha", "lunch": "Chickpea salad", "dinner": "Dal + Roti"},
		})
	if "tailor resume" in prompt.lower():
		return "Tailored text..."
	return "{}"


def google_search_stub(query: str, num_results=3) -> List[Dict[str, str]]:
	"""Placeholder search tool. Return mocked store/product results."""
	log(f"Search: {query}")
	# Return simple mocked results
	return [
		{"title": "Local Grocery Store", "url": "https://example.com/store1", "price_est": "100"},
		{"title": "Supermarket Online", "url": "https://example.com/store2", "price_est": "120"},
	]


# -----------------------------
# Pantry & Planning agents (simple heuristics for the demo)
class PantryAgent:
	def __init__(self, memory: MemoryBank):
		self.memory = memory

	def check_missing(self, plan: Dict[str, Dict[str, str]]) -> Dict[str, float]:
		"""Compute a very small required-ingredient map from the plan and compare to pantry."""
		pantry = self.memory.get_pantry()
		required: Dict[str, float] = {}

		for day, meals in plan.items():
			for meal_name, dish in meals.items():
				dish_lower = (dish or "").lower()
				# tiny example mapping
				if "paneer" in dish_lower:
					required["paneer"] = required.get("paneer", 0) + 200
				if "dal" in dish_lower:
					required["lentils"] = required.get("lentils", 0) + 250
				if "oats" in dish_lower:
					required["oats"] = required.get("oats", 0) + 100
				if "pulao" in dish_lower or "poha" in dish_lower or "vegetable" in dish_lower:
					required["rice"] = required.get("rice", 0) + 200

		missing: Dict[str, float] = {}
		for k, v in required.items():
			if pantry.get(k, 0) < v:
				missing[k] = v - pantry.get(k, 0)

		log(f"Missing items computed: {missing}")
		return missing


class MealPlanningAgent:
	def __init__(self, user_profile: Dict[str, Any]):
		self.user_profile = user_profile

	def plan_week(self) -> Dict[str, Dict[str, str]]:
		prompt = f"Plan meals for: {self.user_profile}"
		resp = llm_generate(prompt)
		try:
			plan = json.loads(resp)
		except Exception:
			# fallback small plan
			plan = {
				"Monday": {"breakfast": "Oats with fruit", "lunch": "Vegetable pulao", "dinner": "Paneer curry"},
				"Tuesday": {"breakfast": "Poha", "lunch": "Chickpea salad", "dinner": "Dal + Roti"},
			}
		return plan


class GroceryAgent:
	def __init__(self):
		pass

	def build_grocery_list(self, missing: Dict[str, float]) -> List[Dict[str, Any]]:
		out: List[Dict[str, Any]] = []
		for item, qty in missing.items():
			search = google_search_stub(f"buy {item}")
			out.append({"item": item, "qty": qty, "options": search})
		log("Grocery list built")
		return out


class ShoppingAgent:
	def __init__(self):
		pass

	def choose_options(self, grocery_list: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
		# choose cheapest of options (based on mocked price_est)
		chosen: List[Dict[str, Any]] = []
		for row in grocery_list:
			opts = row.get("options", [])
			if not opts:
				chosen.append({**row, "chosen": None})
			else:
				opts_sorted = sorted(opts, key=lambda x: float(x.get("price_est", 1e9)))
				chosen.append({**row, "chosen": opts_sorted[0]})
		log("Shopping options chosen")
		return chosen


# -----------------------------
# Orchestrator
# -----------------------------


def run_weekly_pipeline():
	log("Starting weekly pipeline")
	mp_agent = MealPlanningAgent(USER_PROFILE)
	plan = mp_agent.plan_week()

	pa_agent = PantryAgent(memory)
	missing = pa_agent.check_missing(plan)

	ga_agent = GroceryAgent()
	grocery_list = ga_agent.build_grocery_list(missing)

	sa_agent = ShoppingAgent()
	chosen = sa_agent.choose_options(grocery_list)

	# Save plan and outputs
	out = {"plan": plan, "missing": missing, "grocery_list": chosen}
	with open(os.path.join(DATA_DIR, "last_run.json"), "w") as f:
		json.dump(out, f, indent=2)

	memory.append_plan({"timestamp": time.time(), "plan": plan})
	log("Weekly pipeline finished. Outputs saved to data/last_run.json")


if __name__ == "__main__":
	run_weekly_pipeline()
