# python-project
# Core Django Models (models.py)
from django.db import models
from django.contrib.auth.models import User  # Import the User model

class EvaluationTemplate(models.Model):
    """
    Defines the structure of an evaluation form.  Allows for creating
    different evaluation templates for different roles or departments.
    """
    name = models.CharField(max_length=255, verbose_name="Template Name")
    description = models.TextField(blank=True, verbose_name="Template Description")
    created_at = models.DateTimeField(auto_now_add=True)
    is_active = models.BooleanField(default=True, verbose_name="Active Template")

    def __str__(self):
        return self.name

class EvaluationSection(models.Model):
    """
    Divides an evaluation template into sections (e.g., "Core Competencies", "Job-Specific Skills").
    """
    template = models.ForeignKey(EvaluationTemplate, on_delete=models.CASCADE, related_name="sections", verbose_name="Evaluation Template")
    name = models.CharField(max_length=255, verbose_name="Section Name")
    order = models.IntegerField(default=1, verbose_name="Section Order")

    def __str__(self):
        return f"{self.template.name} - {self.name}"
    
    class Meta:
        ordering = ['order']  # Ensure sections are ordered

class EvaluationQuestion(models.Model):
    """
    Defines a question within an evaluation section.
    """
    section = models.ForeignKey(EvaluationSection, on_delete=models.CASCADE, related_name="questions", verbose_name="Evaluation Section")
    question_text = models.TextField(verbose_name="Question Text")
    question_type = models.CharField(
        max_length=20,
        choices=[
            ("rating", "Rating Scale"),
            ("text", "Open-Ended Text"),
            ("yesno", "Yes/No"),
        ],
        default="rating",
        verbose_name="Question Type",
    )
    order = models.IntegerField(default=1, verbose_name="Question Order")
    
    def __str__(self):
        return self.question_text[:50] + "..."  # Show a truncated question

    class Meta:
        ordering = ['order']

class PerformanceEvaluation(models.Model):
    """
    Represents a specific performance evaluation for an employee.
    """
    employee = models.ForeignKey(User, on_delete=models.CASCADE, related_name="employee_evaluations", verbose_name="Employee")
    manager = models.ForeignKey(User, on_delete=models.CASCADE, related_name="manager_evaluations", verbose_name="Manager")
    template = models.ForeignKey(EvaluationTemplate, on_delete=models.CASCADE, verbose_name="Evaluation Template")
    start_date = models.DateField(verbose_name="Evaluation Start Date")
    end_date = models.DateField(verbose_name="Evaluation End Date")
    status = models.CharField(
        max_length=20,
        choices=[
            ("pending", "Pending"),
            ("in_progress", "In Progress"),
            ("completed", "Completed"),
            ("cancelled", "Cancelled"),
        ],
        default="pending",
        verbose_name="Evaluation Status",
    )
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"Evaluation for {self.employee.username} ({self.start_date} - {self.end_date})"

class EvaluationResponse(models.Model):
    """
    Stores an employee's response to a specific evaluation question.
    """
    evaluation = models.ForeignKey(PerformanceEvaluation, on_delete=models.CASCADE, related_name="responses", verbose_name="Performance Evaluation")
    question = models.ForeignKey(EvaluationQuestion, on_delete=models.CASCADE, verbose_name="Evaluation Question")
    response_text = models.TextField(blank=True, verbose_name="Response Text")
    rating = models.IntegerField(blank=True, null=True, verbose_name="Rating")  # For rating scale questions

    def __str__(self):
        return f"Response to {self.question.question_text[:30]}... by {self.evaluation.employee.username}"

class Goal(models.Model):
    """
    Defines a goal for an employee as part of the performance evaluation process.
    """
    employee = models.ForeignKey(User, on_delete=models.CASCADE, related_name="goals", verbose_name="Employee")
    goal_text = models.TextField(verbose_name="Goal Description")
    start_date = models.DateField(verbose_name="Goal Start Date")
    end_date = models.DateField(verbose_name="Goal End Date")
    status = models.CharField(
        max_length=20,
        choices=[
            ("not_started", "Not Started"),
            ("in_progress", "In Progress"),
            ("completed", "Completed"),
            ("on_hold", "On Hold"),
        ],
        default="not_started",
        verbose_name="Goal Status",
    )
    progress = models.IntegerField(default=0, verbose_name="Progress (%)")  # Add a progress percentage
    evaluation = models.ForeignKey(PerformanceEvaluation, on_delete=models.SET_NULL, related_name="goals", blank=True, null=True, verbose_name="Performance Evaluation") #Connect to evaluation
    
    def __str__(self):
        return self.goal_text[:50] + "..."
    
    class Meta:
        ordering = ['end_date'] #order by the end date

class Feedback(models.Model):
    """
    Stores feedback provided by managers, peers, or employees as part of the evaluation process.
    """
    evaluation = models.ForeignKey(PerformanceEvaluation, on_delete=models.CASCADE, related_name="feedback_items", verbose_name="Performance Evaluation")
    feedback_provider = models.ForeignKey(User, on_delete=models.CASCADE, related_name="provided_feedback", verbose_name="Feedback Provider")
    feedback_recipient = models.ForeignKey(User, on_delete=models.CASCADE, related_name="received_feedback", verbose_name="Feedback Recipient")
    feedback_text = models.TextField(verbose_name="Feedback Text")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="Feedback Date")
    is_360 = models.BooleanField(default=False, verbose_name="360 Feedback") #flag for 360 feedback

    def __str__(self):
        return f"Feedback from {self.feedback_provider.username} to {self.feedback_recipient.username} on {self.evaluation}"
    
    class Meta:
        ordering = ['created_at']

