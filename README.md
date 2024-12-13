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
        parameters.EntityLogicalName = "account";
        if (selectedRecordid != null && selectedRecordid != undefined) {
            parameters.EntityId = selectedRecordid.replace("{", "").replace("}", "");
        } else {
            parameters.EntityId = formContext.data.entity.getId().replace("{", "").replace("}", "");
        }

        var CloneRecordRequest = {
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
                    operationName: "bosch_CloneRecordRequest"
                };
            }
        };

        Xrm.WebApi.online.execute(bosch_CloneRecordRequest).then(
            function success(result) {
                if (result.ok) {
                    var results = result.json().then(function (response) {
                        Xrm.Utility.openEntityForm("account", response.clonedRecord);

                    });
                }
            },
            function (error) {
                Xrm.Utility.alertDialog(error.message);
            }
        );
    }
}


# Workflow

Create a Custom Workflow Action  in my case bosch_CloneRecordRequest

![image](https://github.com/user-attachments/assets/998c131b-b1fb-4415-8219-662a1c870e18)

