 

<img src="https://media.giphy.com/media/M1oPO3TchPJS/giphy.gif" width="500px" height="250px"/> 

**_"Loyalty to someone is different from depending on them."_**   
                                             - _Zaraki Kenpachi_

## Our Doctor 
```url 
https://admin.tracelyfe.com/employee/api/doctor-lists/
```
```python 
class DoctorsList(APIView):
    roles = [ORGANISATION_ADMIN, ]
    def get(self, request, *args, **kwargs):
        try:
            org  = request.user.profile.filter(primary=True).only('location').first().location.organisation
            doctor = OrganisationDoctor.objects.get_organisation_doctors(org)
            serializers = EmpDoctorSerializer(doctor, context={"organisation":org},many=True)
            return Response({'status': 'SUCCESS', 'data': serializers.data}, status=200)
        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400)
```

```url
https://admin.tracelyfe.com/employee/api/doctor-all-list
```
```python 
class AllDoctorsList(APIView):
    
    def get(self, request, *args, **kwargs):
        try:
            org  = request.user.profile.filter(primary=True).only('location').first().location.organisation
            doctor = OrganisationDoctor.objects.get_organisation_doctors(org).values_list("id", flat=True)
            doct = Doctor.objects.all().exclude(id__in=doctor)
            serializers = CreateDoctorSerializer(doct, context={"organisation":org}, many=True)
            return Response({'status': 'SUCCESS', 'data': serializers.data}, status=200)
        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400) 
```

```url
https://admin.tracelyfe.com/employee/api/pending-doctor-list/
```
```python 
class PendingDoctorsList(APIView):
    
    def get(self, request, *args, **kwargs):
        try:

            org  = request.user.profile.filter(primary=True).only('location').first().location.organisation
            doctor_org = OrganisationDoctor.objects.get_organisation_doctors(org).values_list("id", flat=True)

            doct = DoctorORGInvite.objects.filter(status=1,doctor__in=doctor_org)
            docc = Doctor.objects.filter(doctororginvite__in=doct)
            serializers = CreateDoctorSerializer(docc, many=True)            
            return Response({'status': 'SUCCESS', 'data': serializers.data}, status=200)

        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400)  
```
```url 
https://admin.tracelyfe.com/doctor/api/doctor-invite/
```
```python 
class DoctorInviteApi(APIView):  
       roles = [ORGANISATION_ADMIN, ]
       def post(self, request, *args, **kwargs):
            try:
                doctor_id = request.data.get('doctor_id')
                org = request.user.profile.first().location.organisation
                doct = Doctor.objects.get(id=doctor_id)
                notification = Notification.objects.create(doctor=doct, types="PRIMARY", org=org)
                docinvitte,_  = DoctorORGInvite.objects.get_or_create(doctor=doct,organisation=org, status=1, invitation=request.data['invitation'],tnc=request.data['tnc'],acknowledge=request.data['acknowledge'])
                print("docinvitte", docinvitte)
                return Response({'status': 'SUCCESS', 'data': "Sucessfully Added!"}, status=200)
            except Exception as e:
                return Response({'status':'ERROR', 'message': str(e)}, status=400)
```
#### then on doctor dashboard we have a socket of doctor_notifications
```url
wss://admin.tracelyfe.com/ws/doctor_notifications/
```
```python
class DoctorNotificationConsumer(AsyncJsonWebsocketConsumer):
    
    async def connect(self):
        self.user = None
        await self.accept()

    async def disconnect(self, code):
        print('disconnected :- ', code)
        
    async def receive_json(self, data):
        try:
            if self.user:
                pass
            else:
                if 'token' in data.keys():
                    token = data['token']
                    authenticated, is_doctor = await self.authenticate_user(token)
                    if authenticated and is_doctor:
                        self.room_name = f'notifications-{self.user.id}'
                        await self.channel_layer.group_add(
                                self.room_name,
                                self.channel_name
                            )   
                    else:
                        await self.close()
                else:
                    await self.close()
        except Exception as e:
            print('eeeee---', e)
            await self.close()
            
    async def broadcast_message(self, event):
        await self.send_json({
            'data': event['data']
        })
                
    @database_sync_to_async    
    def authenticate_user(self, token):
        try:
            auth_class = authentication.JWTAuthentication()
            validated_token = auth_class.get_validated_token(token)
            self.user = auth_class.get_user(validated_token)
            is_doctor = self.user.userrole_set.filter(role__name=DOCTOR).exists()
            return True, is_doctor
        except Exception as e:
            print(e)
```

