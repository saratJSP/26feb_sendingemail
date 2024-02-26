






public class OrderValidationHelper {
    public static void Checking_orders(Order order){
        List<OrderItem> orderItems = [SELECT Id, Product2Id, Quantity, Product2.Inventory__c from OrderItem where OrderId =: order.Id];
        string validation_error = '';
        Boolean isValid = true;
        
        
        for (OrderItem Item :orderItems){
            if (item.Product2.Inventory__c <=  0){
                isValid = false ;
                validation_error += item.Id + 'insufficient stock ';
            }
             if (item.Quantity <=  0) {
                isValid = false;
                validation_error +=  item.Id + '  Quantity must be greater than zero.\n';
            }
            
        }
        order.IsOrderValid__c = isValid;
        order.ValidationErrors__c = validation_error;
        
 
    }
    @future 
    public static void sending_email_notifications(Id orderId){
        Order order = [SELECT Id , Owner.Email,IsOrderValid__c,ValidationErrors__c from Order where Id =: orderId];
        if (! order.IsOrderValid__c){
            
              Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
            String[] toaddresses = new String[] {'saratdhoni7@gmail.com'}; 

            mail.setToAddresses(toaddresses);
            mail.setSubject('Order Validation Status');
            mail.setPlainTextBody('Hi ' + order.Owner.Name + ',\n\n' +
                                  'Your order ' + order.Name + ' is ' + (order.IsOrderValid__c ? 'valid' : 'invalid') + '.\n\n' +
                                  'Validation Errors:\n' + order.ValidationErrors__c);
            Messaging.sendEmail(new Messaging.SingleEmailMessage[] {mail});
        }
        
    }
    

}




trigger Order_validation on Order (before insert ,before update ) {
    
    
    for (Order order : Trigger.new) {
        OrderValidationHelper.Checking_orders(order);
        
        OrderValidationHelper.sending_email_notifications(order.Id);
    
}

}
