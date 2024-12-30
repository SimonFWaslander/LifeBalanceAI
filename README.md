# LifeBalanceAI
from dataclasses import dataclass
from typing import List, Dict, Optional
from datetime import datetime, timedelta
import numpy as np
from enum import Enum

class LifeDomain(Enum):
	CAREER = "career"
	HEALTH = "health"
	FAMILY = "family"
	PERSONAL_GROWTH = "personal_growth"
	SOCIAL = "social"
	FINANCES = "finances"
	HOBBIES = "hobbies"
	SPIRITUALITY = "spirituality"

@dataclass
class DomainMetrics:
	satisfaction: float  # 0-10 scale
	risk: float  # 0-10 scale
	notes: str
	last_updated: datetime

class SALBCalculator:
	def __init__(self, risk_free_rate: float = 5.0):
    	self.risk_free_rate = risk_free_rate
    	self.domains: Dict[LifeDomain, DomainMetrics] = {}
    
	def update_domain(self, domain: LifeDomain, satisfaction: float, risk: float, notes: str = ""):
    	if not (0 <= satisfaction <= 10 and 0 <= risk <= 10):
        	raise ValueError("Satisfaction and risk must be between 0 and 10")
   	 
    	self.domains[domain] = DomainMetrics(
        	satisfaction=satisfaction,
        	risk=risk,
        	notes=notes,
        	last_updated=datetime.now()
    	)
    
	def calculate_salb(self) -> float:
    	if not self.domains:
        	return 0.0
   	 
    	satisfactions = [d.satisfaction for d in self.domains.values()]
    	risks = [d.risk for d in self.domains.values()]
   	 
    	avg_satisfaction = np.mean(satisfactions)
    	avg_risk = np.mean(risks)
   	 
    	# Avoid division by zero
    	if avg_risk == 0:
        	return 0.0
   	 
    	return (avg_satisfaction - self.risk_free_rate) / avg_risk

class LifeBalanceAI:
	def __init__(self):
    	self.salb_calculator = SALBCalculator()
    	self.recommendations: Dict[LifeDomain, List[str]] = {}
    	self.goals: Dict[LifeDomain, List[str]] = {}
   	 
	def update_metrics(self, domain: LifeDomain, satisfaction: float, risk: float, notes: str = ""):
    	"""Update metrics for a specific life domain."""
    	self.salb_calculator.update_domain(domain, satisfaction, risk, notes)
    	self._generate_recommendations(domain)
    
	def _generate_recommendations(self, domain: LifeDomain):
    	"""Generate personalized recommendations based on domain metrics."""
    	metrics = self.salb_calculator.domains.get(domain)
    	if not metrics:
        	return
   	 
    	recommendations = []
   	 
    	# High risk recommendations
    	if metrics.risk > 7:
        	if domain == LifeDomain.CAREER:
            	recommendations.extend([
                	"Consider freelancing or contract work to stabilize income",
                	"Focus on skill development in emerging technologies",
                	"Build professional network through industry events"
            	])
        	elif domain == LifeDomain.HEALTH:
            	recommendations.extend([
                	"Schedule regular check-ins with healthcare providers",
                	"Maintain consistent fasting schedule",
                	"Practice daily mindfulness exercises"
            	])
   	 
    	# Low satisfaction recommendations
    	if metrics.satisfaction < 6:
        	if domain == LifeDomain.SOCIAL:
            	recommendations.extend([
                	"Join community groups aligned with your interests",
                	"Schedule regular social activities",
                	"Explore dating opportunities through shared interest groups"
            	])
        	elif domain == LifeDomain.FINANCES:
            	recommendations.extend([
                	"Create detailed budget for 3D house project",
                	"Explore additional income streams",
                	"Set up emergency fund"
            	])
   	 
    	self.recommendations[domain] = recommendations
    
	def add_goal(self, domain: LifeDomain, goal: str):
    	"""Add a SMART goal for a specific domain."""
    	if domain not in self.goals:
        	self.goals[domain] = []
    	self.goals[domain].append(goal)
    
	def get_salb_score(self) -> float:
    	"""Calculate current SALB score."""
    	return self.salb_calculator.calculate_salb()
    
	def get_domain_summary(self, domain: LifeDomain) -> Dict:
    	"""Get comprehensive summary for a specific domain."""
    	metrics = self.salb_calculator.domains.get(domain)
    	if not metrics:
        	return {}
   	 
    	return {
        	"satisfaction": metrics.satisfaction,
        	"risk": metrics.risk,
        	"notes": metrics.notes,
        	"last_updated": metrics.last_updated,
        	"recommendations": self.recommendations.get(domain, []),
        	"goals": self.goals.get(domain, [])
    	}

@dataclass
class ActionItem:
	description: str
	deadline: datetime
	priority: int  # 1-3, where 1 is highest
	metrics: List[str]
	target_values: List[float]
	current_values: List[float]
	completion_status: float  # 0-1
	notes: str = ""

@dataclass
class MilestoneTracker:
	description: str
	target_date: datetime
	success_criteria: List[str]
	metrics: Dict[str, float]
	achieved: bool = False

class ActionTracker:
	def __init__(self):
    	self.actions: Dict[str, List[ActionItem]] = {}
    	self.milestones: Dict[str, List[MilestoneTracker]] = {}
    	self.progress_history: Dict[str, List[tuple]] = {}  # (datetime, metric, value)
   	 
	def add_action(self, domain: str, action: ActionItem):
    	if domain not in self.actions:
        	self.actions[domain] = []
    	self.actions[domain].append(action)
   	 
	def add_milestone(self, domain: str, milestone: MilestoneTracker):
    	if domain not in self.milestones:
        	self.milestones[domain] = []
    	self.milestones[domain].append(milestone)
   	 
	def update_progress(self, domain: str, action_index: int, metric_index: int, new_value: float):
    	action = self.actions[domain][action_index]
    	action.current_values[metric_index] = new_value
   	 
    	# Calculate completion status
    	total_progress = sum(cv/tv for cv, tv in zip(action.current_values, action.target_values))
    	action.completion_status = total_progress / len(action.metrics)
   	 
    	# Record history
    	if domain not in self.progress_history:
        	self.progress_history[domain] = []
    	self.progress_history[domain].append((datetime.now(), action.metrics[metric_index], new_value))
   	 
	def get_domain_progress(self, domain: str) -> dict:
    	if domain not in self.actions:
        	return {}
       	 
    	total_actions = len(self.actions[domain])
    	completed_actions = sum(1 for action in self.actions[domain]
                            	if action.completion_status >= 0.99)
   	 
    	return {
        	"total_actions": total_actions,
        	"completed_actions": completed_actions,
        	"completion_rate": completed_actions / total_actions if total_actions > 0 else 0,
        	"actions": [
            	{
                	"description": action.description,
                	"completion": action.completion_status,
                	"deadline": action.deadline,
                	"metrics": list(zip(action.metrics, action.current_values, action.target_values))
            	}
            	for action in self.actions[domain]
        	]
    	}

def create_career_action_plan() -> List[ActionItem]:
	return [
    	ActionItem(
        	description="Establish freelance income stream",
        	deadline=datetime.now() + timedelta(days=90),
        	priority=1,
        	metrics=["Monthly income", "Active clients", "Completed projects"],
        	target_values=[5000.0, 3.0, 5.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0,
        	notes="Focus on 3D printing and sustainable architecture projects"
    	),
    	ActionItem(
        	description="Complete 3D printing certification",
        	deadline=datetime.now() + timedelta(days=60),
        	priority=1,
        	metrics=["Courses completed", "Practice hours", "Portfolio projects"],
        	target_values=[5.0, 100.0, 3.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0
    	),
    	ActionItem(
        	description="Build professional network",
        	deadline=datetime.now() + timedelta(days=180),
        	priority=2,
        	metrics=["Industry connections", "Events attended", "Collaborations initiated"],
        	target_values=[50.0, 6.0, 2.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0
    	)
	]

def create_health_action_plan() -> List[ActionItem]:
	return [
    	ActionItem(
        	description="Optimize fasting routine",
        	deadline=datetime.now() + timedelta(days=30),
        	priority=1,
        	metrics=["Fasting hours/week", "Energy level (1-10)", "Mood stability (1-10)"],
        	target_values=[16.0, 8.0, 8.0],
        	current_values=[12.0, 6.0, 7.0],
        	completion_status=0.0
    	),
    	ActionItem(
        	description="Establish stress management routine",
        	deadline=datetime.now() + timedelta(days=45),
        	priority=2,
        	metrics=["Meditation minutes/day", "Anxiety episodes/week", "Sleep quality (1-10)"],
        	target_values=[20.0, 1.0, 8.0],
        	current_values=[5.0, 3.0, 6.0],
        	completion_status=0.0
    	)
	]

def create_financial_action_plan() -> List[ActionItem]:
	return [
    	ActionItem(
        	description="Create 3D house project budget",
        	deadline=datetime.now() + timedelta(days=45),
        	priority=1,
        	metrics=["Budget sections completed", "Cost estimates obtained", "Funding sources identified"],
        	target_values=[10.0, 15.0, 5.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0
    	),
    	ActionItem(
        	description="Build emergency fund",
        	deadline=datetime.now() + timedelta(days=180),
        	priority=1,
        	metrics=["Months of expenses saved", "Monthly saving rate", "Passive income streams"],
        	target_values=[6.0, 0.3, 2.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0
    	)
	]

def create_social_action_plan() -> List[ActionItem]:
	return [
    	ActionItem(
        	description="Expand social network",
        	deadline=datetime.now() + timedelta(days=90),
        	priority=2,
        	metrics=["New connections made", "Events attended", "Follow-up meetings"],
        	target_values=[20.0, 8.0, 5.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0
    	),
    	ActionItem(
        	description="Dating initiative",
        	deadline=datetime.now() + timedelta(days=120),
        	priority=2,
        	metrics=["Dating profiles created", "Dates attended", "Quality connections made"],
        	target_values=[3.0, 8.0, 2.0],
        	current_values=[0.0, 0.0, 0.0],
        	completion_status=0.0
    	)
	]

def main():
	# Initialize tracker
	tracker = ActionTracker()
    
	# Add action plans for each domain
	for domain, plan in [
    	("Career", create_career_action_plan()),
    	("Health", create_health_action_plan()),
    	("Financial", create_financial_action_plan()),
    	("Social", create_social_action_plan())
	]:
    	for action in plan:
        	tracker.add_action(domain, action)
    
	# Add some sample milestones
	tracker.add_milestone(
    	"Career",
    	MilestoneTracker(
        	description="Establish stable income source",
        	target_date=datetime.now() + timedelta(days=90),
        	success_criteria=["Monthly income >= $5000", "At least 3 regular clients"],
        	metrics={"monthly_income": 0, "regular_clients": 0}
    	)
	)
    
	# Example progress update
	tracker.update_progress("Career", 0, 0, 2000.0)  # Update monthly income for first career action
    
	# Print progress report
	for domain in ["Career", "Health", "Financial", "Social"]:
    	progress = tracker.get_domain_progress(domain)
    	print(f"\n{domain} Domain Progress:")
    	print(f"Completion Rate: {progress['completion_rate']*100:.1f}%")
    	print("Current Actions:")
    	for action in progress['actions']:
        	print(f"- {action['description']}: {action['completion']*100:.1f}% complete")
        	for metric, current, target in action['metrics']:
            	print(f"  * {metric}: {current}/{target}")

if __name__ == "__main__":
	main()

