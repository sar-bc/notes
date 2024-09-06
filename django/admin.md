Файл admin.py
====================
from django.contrib import admin
from .models import Order, CommentCrm
from django.utils.safestring import mark_safe   импорт mark_safe (позваляет загрузить пользовательский тэг)

class Comment(admin.StackedInline): # класс для встраивания модели. Из одной модели 
    model = CommentCrm # модель
	# поля для отображения
    fields = ('comment_dt', 'comment_text')
	# поля только для отображение (не редактируемые поля)
    readonly_fields = ('comment_dt',)
	# количество элементов
    extra = 0


class OrderAdm(admin.ModelAdmin):
	# элементы в виде табличного представления наших полей из модели
    list_display = ('id', 'order_status', 'order_name', 'order_phone', 'order_dt', 'get_img')
	# поля которые будут ссылками
    list_display_links = ('id', 'order_name')
	# поиск по этим полям
    search_fields = ('id', 'order_name', 'order_phone', 'order_dt')
	# фильтр по полям
    list_filter = ('order_status',)
	# поля для редактирования не заходя в запись
    list_editable = ('order_status', 'order_phone')
	# поля для отображения в какой то записи, если есть не редактруемые поля (id) то надо readonly_fields
    fields = ('id', 'order_status', 'order_dt', 'order_name', 'order_phone', 'get_img')
	# отобразить не редактируемые поля
    readonly_fields = ('id', 'order_dt')
	# включает данные из разных моделей (Comment это класс который наследуется от admin.StackedInline)
    inlines = [Comment]
	 # метод для отображения миниатюры obj это экземпляр класса модели  Order
	def get_img(self, obj):
       return mark_safe(f"<img src='{obj.cms_img.url}' width='80'>")
	# название поля в админке
    get_img.short_description = 'Миниатюра'

# регистрация модели и класса с настройками для отобажения в админке
admin.site.register(Order, OrderAdm)
