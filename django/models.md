файл models.py
================================
```python
from django.db import models
from django.urls import reverse


class Blog(models.Model):
    title = models.CharField(max_length=255, verbose_name='Название')
    slug = models.SlugField(max_length=255, unique=True, verbose_name='URL')
    content = models.TextField(blank=True, verbose_name='Контент')
    photo = models.ImageField(upload_to="photo/%Y/%m/%d/", blank=True, verbose_name='Фото')
    time_created = models.DateTimeField(auto_now_add=True, verbose_name='Дата публикации')
    time_update = models.DateTimeField(auto_now=True, verbose_name='Дата обновления')
    is_published = models.BooleanField(default=True, verbose_name='Опубликовано')
    cat = models.ForeignKey('Category', on_delete=models.PROTECT, verbose_name='Категория')

    def get_absolute_url(self):
        return reverse('post', kwargs={'post_slug': self.slug})

    def __str__(self):
        return self.title

    class Meta:
        verbose_name = "новость"
        verbose_name_plural = 'новости'
        ordering = ['-time_created']


class Category(models.Model):
    name = models.CharField(max_length=100, verbose_name='Категория')
    slug = models.SlugField(max_length=100, unique=True, verbose_name='URL')

    def get_absolute_url(self):
        return reverse('category', kwargs={'cat_slug': self.slug})

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = "категорию"
        verbose_name_plural = 'категории'
```
Привязка таблиц:
- один ко многим (ManytoOne) ForeignKey
- многие ко многим (ManytoMany) ManyToManyField
- один к одному (OnetoOne) OneToOneField
Класс принимает два обязательных аргумента
1. ссылка или строка класса модели
2. on_delete
опция on_delete="models.CASCADE" - если пользователь будет удален, то удаляется все его записи.
опция on_delete="models.PROTECT" - запрещает удалять пользователя, пока у него есть записи
опция on_delete="models.SET_NULL" - задачи остануться в базе при удалении пользователя, но значение в поле измениться на NULL
опция on_delete="models.SET_DEFAULT" - задачи остануться в базе, при удалении пользователя, по значению в поле измениться на значение по умолчанию
  