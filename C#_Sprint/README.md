# LiveProject for Theatre Site
  	My role in this project was to select user stories and complete them in efficient ways and comment all code 
because it is a very large, collaborative project on a site that will eventually go live. Debugging and running code 
was key to this project because everything has to look a certain way on the site. 
	My user stories involved rental requests(required an understanding of the nuances of time input and using 
the correct formats), production image displays(required linking of files and foreach loops), access to site
information based on user roles, and creating a way to arrange subscribers into a page view that was different from
an html table. 
	
Here a some of the examples of some code I edited in the create and edit views using Html helpers:

//*****************************************Create_View**********************************************************************//

		<div class="form-group">
                    @Html.LabelFor(model => model.StartTime, htmlAttributes: new { @class = "control-label col-md-4 inputLabel" })
                    <div class="col-md-10 formBox">
                        @Html.EditorFor(model => model.StartTime, new { htmlAttributes = new { @class = "form-control", @type = "datetime-local" } })
                        @Html.ValidationMessageFor(model => model.StartTime, "", new { @class = "text-white" }) 			
                    </div>
                </div>

                <div class="form-group">
                    @Html.LabelFor(model => model.EndTime, htmlAttributes: new { @class = "control-label col-md-4 inputLabel" })
                    <div class="col-md-10 formBox">
                        @Html.EditorFor(model => model.EndTime, new { htmlAttributes = new { @class = "form-control", @type = "datetime-local" } })
                        @Html.ValidationMessageFor(model => model.EndTime, "", new { @class = "text-white" }) 
                    </div>
                </div>

		@*Changed to text white because of red background on page*@
		@*Added @type = "datetime-local" to prevent user input error*@

//***************************************************************************************************************************//



//***************************************Edit_View***************************************************************************//

		<div class="form-group">			
                    @Html.LabelFor(model => model.StartTime, htmlAttributes: new { @class = "control-label col-md-4 inputLabel" })
                    <div class="col-md-10 formBox">
                        @Html.EditorFor(model => model.StartTime, new { htmlAttributes = new { @class = "form-control", @type = "datetime-local", 
                       @Value = @Model.StartTime.ToString("yyyy-MM-dd'T'HH:mm") } })
                        @Html.ValidationMessageFor(model => model.StartTime, "", new { @class = "text-white" })
                    </div>
                </div>

                <div class="form-group">
                    @Html.LabelFor(model => model.EndTime, htmlAttributes: new { @class = "control-label col-md-4 inputLabel" })
                    <div class="col-md-10 formBox">                       
                        @Html.EditorFor(model => model.EndTime, new { htmlAttributes = new { @class = "form-control", @type = "datetime-local",
                       @Value = @Model.EndTime.ToString("yyyy-MM-dd'T'HH:mm") } })
                        @Html.ValidationMessageFor(model => model.EndTime, "", new { @class = "text-white" })
                    </div>
                </div>

		@*Time string format for datetime-local: "yyyy-MM-dd'T'HH:mm"*@
		@*Added @Value so the time being edited would auto populate and keep the input format*@

//***************************************************************************************************************************//	  
	
..and some of the examples of some loops I created in the rental request controller to make sure the user would rent for at 
least and hour and wouldn't choose a start time before and end time before their information was entered into the database:

//***************************************************************************************************************************//

		long sec = 10000000;        //DateTime ticks per second
                long hrSecs = 3600;         //Seconds in an hour
                long Strtm = Convert.ToInt64(rentalRequest.StartTime.Ticks) / sec;    //divided ticks into seconds
                long Endtm = Convert.ToInt64(rentalRequest.EndTime.Ticks) / sec;
                if (Endtm < (Strtm + hrSecs) && Endtm >= Strtm)    //Doesn't allow rentals < 1hr
                {
                    ModelState.AddModelError("EndTime", "** Rental must be at least 1 hour.  **");  
                }
                if (Endtm < Strtm)   //Keeps End time after start time
                {
                    ModelState.AddModelError("StartTime", "** Start Time cannot occur after End Time.  **");
                }

		//This was used in the "Create" and "Edit" functions

//***************************************************************************************************************************//

This is a loop I created to display the correct image to all productions that had a default image in the database:

//***************************************************************************************************************************//

		@foreach (var item in Model)
		{
		<div class="card production-admin-div">
    		    @Html.DisplayFor(modelItem => item.DefaultPhoto)
   		    @{
        		if (item.DefaultPhoto != null)
        		{
            		    <img class="card-img-top" src="@Url.Action("DisplayPhoto", "Photo", new { id = item.DefaultPhoto.PhotoId })" alt="Card image cap">
        		}
        		else
        		{
            		    <img class="card-img-top" src="~/Content/Images/Unavailable.png" alt="Card image cap">
        		}
    		    }

		//The Unavailable.png is just a picture I made in paint that says "Photo Unavailable"
		//This is in case the production didn't have a default photo in the database

//***************************************************************************************************************************//

This was my front end fix to make sure the a user not signed in as "Admin" couldn't edit or delete a cast member's
information:

//***************************************************************************************************************************//
				
		<p>
    		    @*Create is admin only*@
    		    @if (User.IsInRole("Admin"))
    		    {
        		@Html.ActionLink("Create New", "Create")
    		    }
		</p>

		<p>
                    @*Delete and edit are admin only*@
                    @if (User.IsInRole("Admin"))
                    {
                        @Html.ActionLink("Edit", "Edit", new { id = item.CastMemberID })
                        @(" |")
                    }
                    @*@Html.ActionLink("Details", "Details", new { id = item.CastMemberID })*@
                    @if (User.IsInRole("Admin"))
                    {
                        @Html.ActionLink("Delete", "Delete", new { id = item.CastMemberID })
                    }
                </p>
		

//***************************************************************************************************************************//

This was my idea to arrange subscriber information using bootstrap elements:

//***************************************************************************************************************************//

                <body class="subbody">
                    <h2 class="subback1">Index</h2>
                    <p>
                        @Html.ActionLink("Create New", "Create")
                        &nbsp;&nbsp;@Html.ActionLink("Back to Dashboard", "../Dashboard/Index")
                    </p>
                    <div class="container-fluid">
                    @foreach (var item in Model)
                    {
                        <div class="subback2" style="padding-top:0; padding-bottom:0;">
                            <div class="row align-items-start accordion" style="padding-bottom:0;">
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.SubscriberPerson.FirstName)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.SubscriberPerson.FirstName, new { htmlAttributes = new { @style = "text-decoration-color:black;" } })
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.SubscriberPerson.LastName)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.SubscriberPerson.LastName)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.SubscriberPerson.UserName)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.SubscriberPerson.UserName)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.CurrentSubscriber)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.CurrentSubscriber)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.HasRenewed)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.HasRenewed)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.Newsletter)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.Newsletter)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.RecentDonor)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.RecentDonor)
                                    </p>
                                </div>
                            </div>
                            <div class="row align-items-start">
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.LastDonated)
                                    <p class="text-body accordion">
                                        @Html.DisplayFor(modelItem => item.LastDonated)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.LastDonationAmt)
                                    <p class="text-body">
                                        @Html.DisplayFor(modelItem => item.LastDonationAmt)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.SpecialRequests)
                                    <p>
                                        @Html.TextAreaFor(modelItem => item.SpecialRequests)
                                    </p>                       
                                </div>
                                <div class="col-auto">
                                    @Html.DisplayNameFor(model => model.Notes)
                                    <p>
                                        @Html.TextAreaFor(modelItem => item.Notes)
                                    </p>
                                </div>
                                <div class="col-auto">
                                    @Html.ActionLink("Edit", "Edit", new { id = item.SubscriberId })<nobr>&nbsp;|</nobr>
                                    @Html.ActionLink("Details", "Details", new { id = item.SubscriberId })<nobr>&nbsp;|</nobr>
                                    @Html.ActionLink("Delete", "Delete", new { id = item.SubscriberId })
                                </div>
                            </div>
                        </div>
                    }
                    </div>
                </body>	 		

//***************************************************************************************************************************//







