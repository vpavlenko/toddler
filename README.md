# Toddler

Childish REST-ignorant way to bootstrap Django APIs

```python
import functools
import inspect
import json

import stringcase

from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from django.views.decorators.http import require_POST


def toddler(view):
    @functools.wraps(view)
    @require_POST
    @csrf_exempt
    def wrapper(request):
        body = json.loads(request.body)
        args = []
        for arg_name in inspect.getargspec(view).args:
            if arg_name == 'request':
                args.append(request)
            else:
                args.append(body.get(stringcase.camelcase(arg_name), None))
        response = view(*args)
        # TODO: Make deep camelization when needed.
        return JsonResponse({stringcase.camelcase(k): v for k, v in view(*args).items()})
    return wrapper
```

@toddler
def get_notes(request):
  return {
    todos: Note.models.filter(user=request.user)
  }
  
@toddler
def save_todo(request, note):
  ...
```  
