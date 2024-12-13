# CloneRecordsDynamically
Clone Record Dynamically

If you are having Custom Entity or Out of the Box table record and you want to clone the same, 

if so you can follow my below steps to achieve the same

#Button :

Create a Button using Ribbon Work Bench

#JavaScript

On Click of the Button call below Script

cloneRecord: function (primaryControl, selectedRecordid) {
        var formContext = primaryControl;

        var parameters = {};
        parameters.EntityLogicalName = "fwc_vpdform";
        if (selectedRecordid != null && selectedRecordid != undefined) {
            parameters.EntityId = selectedRecordid.replace("{", "").replace("}", "");
        } else {
            parameters.EntityId = formContext.data.entity.getId().replace("{", "").replace("}", "");
        }

        var fwc_CloneVPDRecordRequest = {
            EntityLogicalName: parameters.EntityLogicalName,
            EntityId: parameters.EntityId,


            getMetadata: function () {
                return {
                    boundParameter: null,
                    parameterTypes: {
                        "EntityLogicalName": {
                            "typeName": "Edm.String",
                            "structuralProperty": 1
                        },
                        "EntityId": {
                            "typeName": "Edm.String",
                            "structuralProperty": 1
                        }
                    },
                    operationType: 0,
                    operationName: "fwc_CloneVPDRecord"
                };
            }
        };

        Xrm.WebApi.online.execute(fwc_CloneVPDRecordRequest).then(
            function success(result) {
                if (result.ok) {
                    var results = result.json().then(function (response) {
                        Xrm.Utility.openEntityForm("fwc_vpdform", response.clonedRecord);

                    });
                }
            },
            function (error) {
                Xrm.Utility.alertDialog(error.message);
            }
        );
    }
}
