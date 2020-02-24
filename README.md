# LiveProject for Hobby Manager
  My app's purpose in this project was to get user info for the types of surfing for oregon surfers including a choice of four 
coastal outposts for daily tide data, the type of surfboard they prefer, and where they are from. The outpost choice would 
then link to the corresponding NOAA outpost(closest to the beach the user surfs).

  I started with a base.html with two simple buttons: Home and Info. Home would just render a simple page with a the message
"Welcome to the Wave Watch App" and "A resource for surfers looking to get wet in Oregon's best coves and beaches. Daily NOAA
wave data is provided from 4 outposts along Oregon's amazing coastline" Info would bring up collected data and give the option 
to add or edit/delete data. I stared by creating a surftype model using the Django ModelForm module:

//***************************************************************************************************************************
  class BoardForm(ModelForm):
      class Meta:
          model = surfType
          fields = '__all__'
//***************************************************************************************************************************
          
  I then laid out a form for inputting into a database using the Django models module. I tried to be as deliberate as possible with 
character lengths and which inputs would be necessary and how these values would work with future changes: 

//***************************************************************************************************************************
  BOARD_TYPES = (('Long', 'Long'),('short', 'short'),('fish', 'fish'),('egg', 'egg'))

  BOARD_LENGTHS = (("6'", "6'"),("6'-6\"", "6'-6\""),("7'", "7'"),("7'-6\"", "7'-6\""),("8'", "8'"),("8'-6\"", "8'-6\""),
                 ("9'", "9'"),("9'-6\"", "9'-6\""),("10'", "10'"),("10'-6\"", "10'-6\""),("11'", "11'"),("SUP", "SUP"))

  OUTPOSTS = (
      ('Garibaldi', 'Garibaldi'),
      ('SouthBeach', 'South Beach'),
      ('Charleston', 'Charleston'),
      ('PortOrford', 'Port Orford')
  )

  class surfType(models.Model):
      name = models.CharField(max_length=40)
      locale = models.CharField(max_length=40)
      country = models.CharField(max_length=30, blank=True)
      board_type = models.CharField(max_length=20, choices=BOARD_TYPES)
      board_length = models.CharField(max_length=20, choices=BOARD_LENGTHS)
      Oregon_Outpost = models.CharField(max_length=30, choices=OUTPOSTS)

      surfTypes = models.Manager()

      def __str__(self):
          return self.locale
//***************************************************************************************************************************
This would provide the necessary data to use the NOAA data in a meaningful way.

  I then added a views.py file to link to the index and edit pages using my surfType and BoardForm modules and the Django
render, redirect, get_object_or_404, and timezone modules:

//***************************************************************************************************************************
  def WaveHome(request):
      return render(request, "WaveWatch/wavewatch_home.html")

  def index(request):
      get_surfinfo = surfType.surfTypes.all()      #Gets all the current info from the database
      context = {'surfdata': get_surfinfo}       #Creates a dictionary object of all the info for the template
      return render(request, 'WaveWatch/wavewatch_index.html', context)

  def add_surfInfo(request):
      form = BoardForm(request.POST or None)     #Gets the posted form, if one exists
      if form.is_valid():                         #Checks the form for errors, to make sure it's filled in
          form.save()                             #Saves the valid form/jersey to the database
          return redirect('listSurfType')         #Redirects to the index page, which is named 'waves' in the urls
      else:
          print(form.errors)                      #Prints any errors for the posted form to the terminal
          form = BoardForm()                     #Creates a new blank form
      return render(request, 'WaveWatch/wavewatch_create.html', {'form':form})

  def details_surfInfo(request, pk):
      pk = int(pk)                                #Casts value of pk to an int so it's in the proper form
      surfID = get_object_or_404(surfType, pk=pk)   #Gets single instance of the data from the database
      context={'surfID':surfID}                    #Creates dictionary object to pass the object
      return render(request,'WaveWatch/wavewatch_details.html', context)

  def update_surfInfo(request, pk):
      surfID = get_object_or_404(surfType, pk=pk)
      if request.method == "POST":
          form = BoardForm(request.POST, instance=surfID)
          if form.is_valid():
              surfID = form.save(commit=False)
              surfID.author = request.user
              surfID.published_date = timezone.now()
              surfID.save()
              return redirect('surfInfoDetails', pk=surfID.pk)
      else:
          form = BoardForm(instance=surfID)
      return render(request, 'WaveWatch/wavewatch_update.html', {'form': form})

  def delete_surfInfo(request, pk):
      surfID = get_object_or_404(surfType, pk=pk)
      #POST request
      if request.method == "POST":
          #confirming delete
          surfID.delete()
          return redirect('../../')
      context = {"surfID": surfID}
      return render(request, "WaveWatch/wavewatch_delete.html", context)
//***************************************************************************************************************************      

*These are all linked with the following views in my urls.py file:

//***************************************************************************************************************************
  urlpatterns = [
    path('', views.WaveHome, name='waves'),
    path('Collection/', views.index, name='listSurfType'),  # index of jerseys
    path('AddTo/', views.add_surfInfo, name='addSurfInfo'),  # index of jerseys
    path('Collection/<int:pk>/Details/', views.details_surfInfo, name='surfInfoDetails'),
    path('Collection/<int:pk>/Update/', views.update_surfInfo, name='updateSurfInfo'),
    path('Collection/<int:pk>/Delete/', views.delete_surfInfo, name='deleteSurfInfo'),
    #path('API/', views.api_response, name='apiInfo'),
  ]
//*************************************************************************************************************************** 

*As the timeframe of the project before I could link to the NOAA APIs, I plan to revisit aspects of this project*



  
