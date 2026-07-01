# ⚡ Performance Tuning Guide

## Database Optimization

### Indexes
```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=255, db_index=True)
    slug = models.SlugField(unique=True, db_index=True)
    category = models.ForeignKey(Category, on_delete=models.CASCADE, db_index=True)
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    
    class Meta:
        indexes = [
            models.Index(fields=['category', 'created_at']),
            models.Index(fields=['name', 'category']),
        ]
```

### Query Optimization
```python
# ❌ Bad: N+1 queries
products = Product.objects.all()
for product in products:
    print(product.category.name)  # Query per product

# ✅ Good: Select related
products = Product.objects.select_related('category')
for product in products:
    print(product.category.name)  # No additional queries
```

## Caching Strategy

### Redis Caching
```python
from django.views.decorators.cache import cache_page
from django.core.cache import cache

# View-level caching
@cache_page(60 * 5)  # 5 minutes
def product_list(request):
    return Product.objects.all()

# Manual caching
def get_featured_products():
    products = cache.get('featured_products')
    if products is None:
        products = Product.objects.filter(featured=True)
        cache.set('featured_products', products, 60 * 60)  # 1 hour
    return products
```

## API Response Optimization

### Pagination
```python
from rest_framework.pagination import PageNumberPagination

class ProductListView(ListAPIView):
    queryset = Product.objects.all()
    pagination_class = PageNumberPagination
```

### Response Compression
```nginx
# nginx.conf
gzip on;
gzip_vary on;
gzip_min_length 1000;
gzip_proxied any;
gzip_types text/plain text/css text/json application/json;
```

## Async Tasks

### Celery Configuration
```python
# settings.py
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'
CELERY_TASK_TRACK_STARTED = True
CELERY_TASK_TIME_LIMIT = 30 * 60
```

## Load Testing

### Apache Bench
```bash
# 1000 requests, 10 concurrent
ab -n 1000 -c 10 http://localhost:8000/api/v1/products/

# With keep-alive
ab -k -n 1000 -c 10 http://localhost:8000/api/v1/products/
```

## Monitoring

### Metrics to Track
- Response time (p50, p95, p99)
- Throughput (requests/sec)
- Error rate
- Database query time
- Cache hit rate
- Memory usage
- CPU usage

### Tools
- New Relic
- DataDog
- Prometheus + Grafana
- Sentry (errors)

## Scaling Strategy

### Horizontal Scaling
```bash
# Scale web workers
docker-compose up -d --scale web=3
```

### Vertical Scaling
- Increase gunicorn workers
- Increase memory allocation
- Upgrade CPU

## Checklist
- [ ] Database indexes configured
- [ ] Query optimization done
- [ ] Caching strategy implemented
- [ ] Pagination added
- [ ] Compression enabled
- [ ] Async tasks configured
- [ ] Load testing completed
- [ ] Monitoring configured
- [ ] Auto-scaling enabled
