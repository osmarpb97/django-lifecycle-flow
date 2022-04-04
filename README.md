# Django Lifecycle Hooks Flow

[![PyPI version](https://badge.fury.io/py/django-lifecycle-flow.svg)](https://badge.fury.io/py/django-lifecycle-flow)
![PyPI - Django Version](https://img.shields.io/pypi/djversions/django-lifecycle)

This project provides a `@hook` decorator as well as a base model and mixin to add lifecycle hooks flow to your Django models. Django's built-in approach to offering lifecycle hooks flow is [Signals](https://docs.djangoproject.com/en/dev/topics/signals/). However, my team often finds that Signals introduce unnecessary indirection and are at odds with Django's "fat models" approach.

**Django Lifecycle Hooks Flow** supports Python 3.5, 3.6, 3.7, 3.8 and 3.9, Django 2.0.x, 2.1.x, 2.2.x, 3.0.x, 3.1.x, and 3.2.x.

In short, you can write model code like this:

```python
from django_lifecycle_flow import LifecycleModel, hook, BEFORE_UPDATE, AFTER_UPDATE


class Article(LifecycleModel):
    contents = models.TextField()
    updated_at = models.DateTimeField(null=True)
    status = models.ChoiceField(choices=['draft', 'published'])
    editor = models.ForeignKey(AuthUser)

    @hook(BEFORE_UPDATE, when='contents', has_changed=True)
    def on_content_change(self):
        self.updated_at = timezone.now()

    @hook(AFTER_UPDATE, when="status", was="draft", is_now="published")
    def on_publish(self):
        send_email(self.editor.email, "An article has published!")
```

Instead of overriding `save` and `__init__` in a clunky way that hurts readability:

```python
    # same class and field declarations as above ...

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._orig_contents = self.contents
        self._orig_status = self.status


    def save(self, *args, **kwargs):
        if self.pk is not None and self.contents != self._orig_contents:
            self.updated_at = timezone.now()

        super().save(*args, **kwargs)

        if self.status != self._orig_status:
            send_email(self.editor.email, "An article has published!")
```

---

**Documentation**: <a href="https://osmarpb97.github.io/django-lifecycle-flow/" target="_blank">https://rsinger86.github.io/django-lifecycle</a>

# Testing

Tests are found in a simplified Django project in the `/tests` folder. Install the project requirements and do `./manage.py test` to run them.

# License

See [License](LICENSE.md).
