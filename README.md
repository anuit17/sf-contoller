public class OpportunityEditController {
    public opportunity opp {get; set;}
    private apexpages.standardcontroller sc;
    public boolean showSave{get;set;}
    public OpportunityEditController(apexpages.standardcontroller sc){
        this.sc = sc;
        showSave=false;
    }
    
    public pageReference SaveAction(){
        pageReference detailpage;
        opp = (opportunity)sc.getrecord();
        
        if(opp.stageName == 'Closed Won' || opp.stageName == 'Shortlisted' ){
            Opp_Bid_settings__c opst = Opp_Bid_settings__c.getOrgDefaults();
            opst.Bid_Status_check__c=true;
            update opst;
            return null;
        } else {
            try{
                detailpage = save();
                if(detailpage != null){
                    List<Opportunity_Bid__c> opbidlst=[select Id, Name, ProjectName__c, Term__c,Bid_Status__c, PPA_Price_Submitted__c, Opportunity__r.StageName from Opportunity_Bid__c Where Opportunity__c = :opp.Id];
                    for(Opportunity_Bid__c opbid : opbidlst)
                    {
                        if(opp.stageName == 'Closed Lost'){
                            if(opbid.Bid_Status__c == 'Shortlisted')
                                opbid.Bid_Status__c = 'Shortlisted Lost';
                            else
                                opbid.Bid_Status__c = 'Not Selected';
                        }
                        if(opp.stageName == 'Cancelled')
                            opbid.Bid_Status__c = 'Cancelled';
                        if(opp.stageName == 'Shortlist Lost')
                            if(opbid.Bid_Status__c == 'Shortlisted')
                            opbid.Bid_Status__c = 'Shortlisted Lost';
                        if(opp.stageName == 'Lost Negotiation')
                            opbid.Bid_Status__c = 'Lost Negotiation';
                        
                        if(opp.stageName == 'Passed / Dead')
                            opbid.Bid_Status__c = 'Passed / Dead';
                        if(opp.stageName == 'Won Cancelled')
                        {
                            if(opbid.Bid_Status__c == 'Closed Won')
                                opbid.Bid_Status__c = 'Won Cancelled';
                            else
                                opbid.Bid_Status__c = 'Not Selected';
                        }
                    }
                    update opbidlst;
                }
            }catch(exception ex)
            {
                ApexPages.addMessages(ex);
            }
        }
        return detailpage;
    }
    
    public PageReference Save(){
        PageReference detailPage = sc.save();
        return detailPage;
    }
    public PageReference getprobability()
    {
        opp = (opportunity)sc.getrecord();
        if(opp.StageName == 'Pending Proposal' || opp.StageName == 'Pending RFP')
            opp.Probability=5;
        if(opp.StageName == 'Won Cancelled')
            opp.Probability=0;
        Opportunity OldOpp=[select id,StageName from Opportunity where id =: opp.Id];
        if(OldOpp.StageName == 'Closed Won' && opp.StageName != 'Won Cancelled'){
            ApexPages.addmessage(new ApexPages.message(ApexPages.severity.ERROR,'You can select only Won Cancelled at this stage.'));
        	showSave=true;
        }
        else
        {
            showSave=false;
        }
        
        return null;
    }
}
