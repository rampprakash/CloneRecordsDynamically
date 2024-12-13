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



# Custom Workflow


using Microsoft.Xrm.Sdk.Query;
using Microsoft.Xrm.Sdk.Workflow;
using Microsoft.Xrm.Sdk;
using System;
using System.Activities;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace WorkflowActivity
{
    public class CloneVPDRecord : CodeActivity
    {
        [Input("EntityId")]
        public InArgument<string> EntityId
        {
            get; set;
        }

        [Input("EntityLogicalName")]
        public InArgument<string> EntityLogicalName
        {
            get; set;
        }

        [Output("clonedRecord")]
        public OutArgument<string> clonedRecordOut
        {
            get; set;
        }

        protected override void Execute(CodeActivityContext executionContext)
        {
            //Create the tracing service
            ITracingService tracingService = executionContext.GetExtension<ITracingService>();
            //Create the context
            IWorkflowContext context = executionContext.GetExtension<IWorkflowContext>();
            IOrganizationServiceFactory serviceFactory = executionContext.GetExtension<IOrganizationServiceFactory>();
            IOrganizationService service = serviceFactory.CreateOrganizationService(context.UserId);
            Guid entityGuid = new Guid(this.EntityId.Get(executionContext));
            tracingService.Trace(entityGuid.ToString());
            string entityLogicalName = this.EntityLogicalName.Get(executionContext).ToString();
            Entity originalRecord = Common.retrieve(entityLogicalName, entityGuid, new ColumnSet(true), service);
            Guid clonedRecord = Common.cloneRecord(originalRecord, service, tracingService);
            tracingService.Trace(clonedRecord.ToString());
            clonedRecordOut.Set(executionContext, clonedRecord.ToString());
        }
    }
}

using Microsoft.Xrm.Sdk.Query;
using Microsoft.Xrm.Sdk;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Plugins.Shared
{
    public class Common
    {
        public static Entity retrieve(string entityName, Guid entityId, ColumnSet columns, IOrganizationService service)
        {
            return service.Retrieve(entityName, entityId, columns);
        }

        public static Guid cloneRecord(Entity originalRecord, IOrganizationService service, ITracingService trace)
        {
            Entity clonedRecord = new Entity(originalRecord.LogicalName);
            // ignore formname
            foreach (var attribute in originalRecord.Attributes)
            {
                if (attribute.Key != originalRecord.LogicalName + "id" &&
                    attribute.Key != "createdon" &&
                    attribute.Key != "createdby" &&
                    attribute.Key != "modifiedon" &&
                    attribute.Key != "modifiedby")
                {
                    if (originalRecord.Attributes.Contains(attribute.Key))
                    {
                        clonedRecord[attribute.Key] = attribute.Value;
                    }
                }
            }
            Guid clonedRecordId = service.Create(clonedRecord);
            return clonedRecordId;
        }
    }
}

