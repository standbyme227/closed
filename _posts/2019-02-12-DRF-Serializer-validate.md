---
layout: post
title: [DRF] serializer's validate
tags: [Django, 장고, Serializer, validate, validation, DRF]
---

###Validate

#### 1. 개요
> Django의 View에서 Data의 Validate 적용이 무시되는 경향을 확인  
좀 더 안전한 validate을 위해서 serializer 내부에서 적용시켜봄.

#### 2. 예전 코드
**1. View**
```
class SomethingCreateView(generics.CreateAPIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )
    serializer_class = VPSerializer
    queryset = vp.objects.all()
    
    def create(self, request, *args, **kwargs):
        user = request.user
        a_id = request.data['a']
        
        vp = self.get_queryset()
        
        user_vp = vp.filter(user=user, a_id=a_id)
        
        # Validate 시도        
        if user_vp.count()<0:
            data = 'vp is not valid'
            return Response(serializer.data, status=status.HTTP_409_CONFLICT)
            
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid(raise_exception=True):
        .
        .
        .
         
```

**2. Serializer**

```
class VPSerializer(serializers.ModelSerializer):
    class Meta:
        model = VideoProgress
        fields = "__all__" 
```

#### 3. 문제

> validate 후 기대와 달리, 겹치는 objects의 생성을 막는데 실패하고  
DB로 접근해서 duplicated error를 생성해냄

#### 4. 수정

**1. View**

```
class SomethingCreateView(generics.CreateAPIView):
    permission_classes = (
        permissions.IsAuthenticated,
    )
    serializer_class = VPSerializer
    
    def create(self, request, *args, **kwargs):
        user = request.user 
        
        serializer = self.get_serializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            .
            .
            .
```

**2. Serializer**

```
class VPSerializer(serializers.ModelSerializer):
    class Meta:
        model = VP
        fields = "__all__" 
        
    def validate(self, data):
        user = data['user']
        a_id = data['a']
        
        vp = VP.objects.filter(user=user, a_id=a_id)
        
        if len(video_progress) > 0:
            raise serializers.ValidationError("VP already exist.")
        return data
```


#### 5. 결과

> Front에 오류가 있는것을 확인하고 처리중이다.

> 나는 단지 좀 더 공부를 했을뿐... ㅋㅋㅋ