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
from django.contrib.auth.models import User


def toddler(view):
    @functools.wraps(view)
    @require_POST
    @csrf_exempt
    @sensitive_post_parameters('password')
    def wrapper(request):
        body = json.loads(request.body)
        args = []
        for arg_name in inspect.getargspec(view).args:
            if arg_name == 'request':
                args.append(request)
            else:
                args.append(body.get(stringcase.camelcase(arg_name), None))
        response = view(*args)
        return JsonResponse({stringcase.camelcase(k): v for k, v in view(*args).items()})
    return wrapper


@toddler
def signup(request, email, first_name, last_name, password):
    if not User.objects.filter(username=username).exists():
        user = User.objects.create_user(
            username=email, email=email, first_name=first_name, last_name=last_name, password=password)
        user.save()

        user = authenticate(username=username, password=password)
        return {'status': 'user_created'}
    else:
        user = authenticate(username=username, password=password)
        return {'status': 'successful_authentication'}
