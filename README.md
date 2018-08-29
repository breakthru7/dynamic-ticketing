# dynamic-ticketing

Google map integration to populate dropdown and get distance for price calculation 

````

        $("#ToLocation").autocomplete({
            //source: defaultloclist,

            source: function (request, response) {

                request.term = request.term.toLowerCase();

                //filter defaultloclist manually
                var filtered = [];
                for (var i = 0; i < defaultloclist.length; i++) {

                    //check for shortforms

                    if (request.term == "jln" && defaultloclist[i].label.toLowerCase().indexOf("jalan") !== -1) {

                        filtered.push(defaultloclist[i]);
                    }
                    else if (request.term == "sg" && defaultloclist[i].label.toLowerCase().indexOf("sungai") !== -1) {

                        filtered.push(defaultloclist[i]);
                    }

                    if (request.term == "" || defaultloclist[i].label.toLowerCase().indexOf(request.term) !== -1) {
                        filtered.push(defaultloclist[i]);
                    }
                }

                //find Google prediction if search term not empty
                if (request.term != "") {

                    var ac = new google.maps.places.AutocompleteService();

                    ac.getPlacePredictions({ input: request.term, componentRestrictions: { country: 'my' } },

                    function (predictions, status) {

                        var r = [];

                        //push predictions to temporary array if found , and then return response of concat of local data and predictions
                        if (predictions) {
                            for (var i = 0; i < predictions.length; ++i) {
                                r.push({
                                    label: predictions[i].structured_formatting.main_text + ' (GoogleMap)*',
                                    lat: 0,
                                    lng: 0,
                                    distance: 0,
                                    distance_awana: 0,
                                    ref: predictions[i].reference,
                                });

                            }
                        }
                        response(filtered.concat(r));
                    })
                }
                else {

                    response(filtered);
                }

            },

            select: function (a, b) {

                //for local data
                if (b.item.lat != 0 && b.item.lng != 0) {
                    var LatLng = new google.maps.LatLng(b.item.lat, b.item.lng);

                    map = new google.maps.Map(document.getElementById('map'), {
                        zoom: 13,
                        center: LatLng
                    });

                    var marker = new google.maps.Marker({
                        position: LatLng,
                        map: map,
                        title: b.item.value
                    });

                    marker.setMap(map);

                    //update hidden fields
                    $("#ToLatitude").val(b.item.lat);
                    $("#ToLongitude").val(b.item.lng);
                    $("#ToDistance").val(b.item.distance);
                    $("#ToDistanceAwana").val(b.item.distance_awana);

                    resetForm();
                }
                else {

                    resetForm();

                    var placeservice = new google.maps.places.PlacesService($('#map').get(0));

                    //call google place service
                    if (b.item.ref != "") {
                        placeservice.getDetails({ reference: b.item.ref },
                        function (place, status) {

                            if (place != null) {
                                glat = place.geometry.location.lat();
                                glng = place.geometry.location.lng();

                                var LatLng = new google.maps.LatLng(glat, glng);

                                map = new google.maps.Map(document.getElementById('map'), {
                                    zoom: 13,
                                    center: LatLng
                                });

                                var marker = new google.maps.Marker({
                                    position: LatLng,
                                    map: map,
                                    title: b.item.value
                                });

                                marker.setMap(map);

                                //update hidden fields
                                $("#ToLatitude").val(glat);
                                $("#ToLongitude").val(glng);
                                $("#ToDistance").val(b.item.distance);
                                $("#ToDistanceAwana").val(b.item.distance_awana);

                            }
                        });
                    }

                }

            },
            minLength: 0,
            scroll: true

        }).focus(function () {
            var searchval = $("#ToLocation").val();

            $(this).autocomplete("search", searchval);
        });
````
