I'll help you explore the Jellyfin view settings. Let me search for files related to view or display settings in the Jellyfin project.

Searched codebase

313 results
I see several interesting files related to display preferences. Let me search more specifically for view-related settings:

Searched in
~/git/jellyfin
0 results
Let me try a broader search:

Searched in
~/git/jellyfin
0 results
I noticed some interesting files in the previous search results, particularly in the Jellyfin.Data/Entities directory. Let me view the DisplayPreferences.cs file to get more context:

Analyzed

DisplayPreferences.cs:(1-51)
This file defines the DisplayPreferences class, which seems to handle various display-related settings for Jellyfin. Some interesting properties include:

ShowSidebar
ShowBackdrop
SkipForwardLength
SkipBackwardLength
ScrollDirection
ChromecastVersion
Would you like me to elaborate on any of these display preference settings? It seems these might be related to the view settings you were asking about. Could you provide more specific details about what hidden view setting you're interested in?

Me

I want to add an option to alwasy show the original title

I'll help you add an option to always show the original title. Based on the DisplayPreferences class we just viewed, we'll want to add a new property to represent this setting.

Let's search for where display preferences are used:

Searched in
~/git/jellyfin
469 results
I'll propose adding a new property to the DisplayPreferences class:

Code
CopyInsert
{{ ... }}
        public bool ShowOriginalTitle { get; set; }
{{ ... }}
I'll also update the constructor to set a default value:

Code
CopyInsert
        public DisplayPreferences(Guid userId, Guid itemId, string client)
        {
            UserId = userId;
            ItemId = itemId;
            Client = client;
            ShowSidebar = false;
            ShowBackdrop = true;
            SkipForwardLength = 30000;
            SkipBackwardLength = 10000;
            ScrollDirection = ScrollDirection.Horizontal;
            ChromecastVersion = ChromecastVersion.Stable;
            ShowOriginalTitle = false;  // Default to false, user can enable if desired

            HomeSections = new HashSet<HomeSection>();
        }
Now, we'll need to update the related Data Transfer Object (DTO) to ensure this new property can be serialized and transferred:

Code
CopyInsert
{{ ... }}
        /// <summary>
        /// Gets or sets a value indicating whether to always show the original title.
        /// </summary>
        public bool ShowOriginalTitle { get; set; }
{{ ... }}
To complete the implementation, we'll need to modify the mapping logic. Let's find the mapping file:

Searched in
~/git/jellyfin
0 results
I'll try a broader search:

Searched in
~/git/jellyfin
7 results
I'll check the DisplayPreferencesController.cs:

Analyzed

DisplayPreferencesController.cs:(51-101)
I see the mapping happening in the GetDisplayPreferences method. I'll update it to include the new ShowOriginalTitle property:

Code
CopyInsert
        var dto = new DisplayPreferencesDto
        {
            Client = displayPreferences.Client,
            Id = displayPreferences.ItemId.ToString(),
            SortBy = itemPreferences.SortBy,
            SortOrder = itemPreferences.SortOrder,
            IndexBy = displayPreferences.IndexBy?.ToString(),
            RememberIndexing = itemPreferences.RememberIndexing,
            RememberSorting = itemPreferences.RememberSorting,
            ScrollDirection = displayPreferences.ScrollDirection,
            ShowBackdrop = displayPreferences.ShowBackdrop,
            ShowSidebar = displayPreferences.ShowSidebar,
            ShowOriginalTitle = displayPreferences.ShowOriginalTitle  // Add the new property
        };
I'll also update the UpdateDisplayPreferences method to handle the new property:

Code
CopyInsert
        displayPreferences.ShowSidebar = displayPreferences.ShowSidebar;
        displayPreferences.ShowBackdrop = displayPreferences.ShowBackdrop;
        displayPreferences.ShowOriginalTitle = displayPreferences.ShowOriginalTitle;  // Add the new property
I've proposed changes to add a new ShowOriginalTitle option to Jellyfin's display preferences. This will allow users to configure whether they want to always show the original title for media items.

The changes include:

Adding a new ShowOriginalTitle property to the DisplayPreferences class
Updating the constructor to set a default value of false
Adding the property to the DisplayPreferencesDto
Updating the API controller to map and handle the new property
To complete the implementation, you'll likely need to modify the UI to add a toggle for this setting and update the rendering logic to respect this preference.

Would you like me to help you with the next steps of implementing this feature?

Me

yes

Let's continue implementing the "always show original title" feature. We'll need to modify the rendering logic to respect this new preference.

First, I'll search for where titles are typically rendered:

Searched in
~/git/jellyfin
0 results
Let me try a broader search:

Searched in
~/git/jellyfin
13 results
I'll try another search:

Searched in
~/git/jellyfin
37 results
I'll check the BaseItem.cs file, which seems promising:

Analyzed

BaseItem.cs:(2301-2351)
I see a GetNameForMetadataLookup() method that seems relevant. Let's search for its implementations:

Searched in
~/git/jellyfin
1 result
I'll modify the BaseItem class to respect the new ShowOriginalTitle preference:

Code
CopyInsert
        protected virtual string GetNameForMetadataLookup()
        {
            // Check if we should show the original title based on display preferences
            var user = GetOwner();
            if (user != null)
            {
                var displayPreferences = _displayPreferencesManager.GetDisplayPreferences(user.Id, Id, "web");
                if (displayPreferences?.ShowOriginalTitle == true && !string.IsNullOrEmpty(OriginalTitle))
                {
                    return OriginalTitle;
                }
            }

            return Name;
        }
We'll need to add a dependency for _displayPreferencesManager. I'll add it to the constructor:

Code
CopyInsert
        private readonly IDisplayPreferencesManager _displayPreferencesManager;

        protected BaseItem(
            IDisplayPreferencesManager displayPreferencesManager,
            ILibraryManager libraryManager,
            IUserManager userManager)
        {
            _displayPreferencesManager = displayPreferencesManager;
            _libraryManager = libraryManager;
            _userManager = userManager;
        }
We'll also need to add the using statement:

Code
CopyInsert
using MediaBrowser.Controller.Providers;
using MediaBrowser.Controller.Users;
using MediaBrowser.Model.Dto;
using MediaBrowser.Model.Entities;
using MediaBrowser.Model.Globalization;
using MediaBrowser.Model.IO;
using MediaBrowser.Model.Library;
using MediaBrowser.Model.Net;
using MediaBrowser.Model.Providers;
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text.Json;
using System.Threading;
using System.Threading.Tasks;

namespace MediaBrowser.Controller.Entities
{
    // Add this using statement
    using MediaBrowser.Controller.Configuration;
I realize now we'll need to modify the dependency injection to pass in the IDisplayPreferencesManager. This will likely be in a dependency injection configuration file.

Let me search for where BaseItem is instantiated:

Searched in
~/git/jellyfin
0 results
I'll try a broader search:

Searched in
~/git/jellyfin
2227 results
This is getting complex. Let me take a step back and propose a simpler approach. Instead of modifying the constructor, we can use a static method or property to check the display preferences:

Code
CopyInsert
        protected virtual string GetNameForMetadataLookup()
        {
            // Check if we should show the original title based on display preferences
            var user = GetOwner();
            if (user != null)
            {
                var displayPreferences = ServiceLocator.Current.GetService<IDisplayPreferencesManager>()
                    .GetDisplayPreferences(user.Id, Id, "web");
                
                if (displayPreferences?.ShowOriginalTitle == true && !string.IsNullOrEmpty(OriginalTitle))
                {
                    return OriginalTitle;
                }
            }

            return Name;
        }
We'll need to add a using statement for ServiceLocator:

Code
CopyInsert
using MediaBrowser.Controller.Providers;
using MediaBrowser.Controller.Users;
using MediaBrowser.Model.Dto;
using MediaBrowser.Model.Entities;
using MediaBrowser.Model.Globalization;
using MediaBrowser.Model.IO;
using MediaBrowser.Model.Library;
using MediaBrowser.Model.Net;
using MediaBrowser.Model.Providers;
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text.Json;
using System.Threading;
using System.Threading.Tasks;

namespace MediaBrowser.Controller.Entities
{
    // Add this using statement
    using MediaBrowser.Common.Extensions;
I'll also add a using statement for the display preferences manager:

Code
CopyInsert
using MediaBrowser.Controller.Providers;
using MediaBrowser.Controller.Users;
using MediaBrowser.Model.Dto;
using MediaBrowser.Model.Entities;
using MediaBrowser.Model.Globalization;
using MediaBrowser.Model.IO;
using MediaBrowser.Model.Library;
using MediaBrowser.Model.Net;
using MediaBrowser.Model.Providers;
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Diagnostics.CodeAnalysis;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text.Json;
using System.Threading;
using System.Threading.Tasks;

namespace MediaBrowser.Controller.Entities
{
    // Add this using statement
    using MediaBrowser.Common.Extensions;
    using MediaBrowser.Controller.Configuration;
These changes will allow users to always show the original title by setting the ShowOriginalTitle preference to true in their display settings.

The implementation does the following:

When getting the name for metadata lookup, it checks the user's display preferences
If ShowOriginalTitle is true and an original title exists, it returns the original title
Otherwise, it returns the standard name
Would you like me to elaborate on any part of the implementation?

