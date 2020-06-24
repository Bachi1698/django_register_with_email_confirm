# # Créer un token.py et à l'interieur y mettre
```python 

from django.contrib.auth.tokens import PasswordResetTokenGenerator
from django.utils import six


class TokenGenerator(PasswordResetTokenGenerator):
    def _make_hash_value(self, user, timestamp):
        return (
            six.text_type(user.pk) + six.text_type(timestamp) +
            six.text_type(user.is_active)
        )
account_activation_token = TokenGenerator()

```
# # Dans le setting.py
 ```python
 EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
 ```
 
 # # Dans le views.py
 - dans la fonction qui s'occupe de l'inscription mettre is_active = False
 ```python
 user = User(
                    # avec les autres élements à enregistrer
                    is_active = False,
                )
 ```
- créer une fonction qui va s'occuper de vérifier les données passées dans url 

```python
def activate(request, uidb64, token):
    
    try:
        uid = force_text(urlsafe_base64_decode(uidb64))
        user = User.objects.get(pk=uid)
        print(token)
        print(user)
    except(TypeError, ValueError, OverflowError, User.DoesNotExist):
        user = None
    if user is not None and account_activation_token.check_token(user, token):
        user.is_active = True
        user.save()
        datas = {
            'confirmation': True,
            'is_actif': True,
            'message': "Votre email a été V"
        }
        return render(request, 'email_confirm.html', datas)
    else:
        return render(request, 'invalid_link.html')
        
        ```
 
  
  # # Dans url.py
  
  ```python
  path('account_confirm/<slug:uidb64>/<slug:token>/',views.activate,name="account_confirm_email")
  ```


