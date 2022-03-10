 

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
