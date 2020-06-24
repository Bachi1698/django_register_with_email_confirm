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
 - importer 
 ```python
######### email conf
from django.contrib.sites.shortcuts import get_current_site
from django.utils.encoding import force_bytes, force_text
from django.utils.http import urlsafe_base64_encode, urlsafe_base64_decode
from django.template.loader import render_to_string
from .token import account_activation_token
from django.core.mail import send_mail
`````
 - dans la fonction qui s'occupe de l'inscription mettre is_active = False
 ```python
 user = User(
                    # avec les autres élements à enregistrer
                    is_active = False,
                )
 ```
- après l'enregistrement du mot de passe 
```python
### email conf
     
                current_site = get_current_site(request) # permet de recuperer le site courant 
                mail_subject = 'Activate your blog account.' # le sujet du mail
                # permet de faire le rendu du mail avec des variable
                message = render_to_string('mail_conf.html', {
                    'user': user,
                    'domain': current_site.domain,
                    'uid':urlsafe_base64_encode(force_bytes(user.pk)),
                    'token':account_activation_token.make_token(user),
                })
                # permet d'envoyer le mail il faut signifier dans ke site configuration smpt
                send_mail(
                        mail_subject,
                        message,
                        'marylise@gmail.com',
                        [user.email],
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
        
 ````



# # Dans url.py
```python
  path('account_confirm/<slug:uidb64>/<slug:token>/',views.activate,name="account_confirm_email")
  ```
  
# #  html 
- Créer des fichiers email_confirm.html , invalid_link.html ,mail_conf.html
- Dans le fichier mail_conf.html y mettre
```python
{% autoescape off %} Hi {{ user.username }}, Please click on the link to confirm
your registration, {{ domain }}{% url 'account_confirm_email' uidb64=uid token=token %}
{% endautoescape %}
```

