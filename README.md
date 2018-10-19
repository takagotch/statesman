### stateman
---
.rb
https://github.com/gocardless/statesman/
https://rubygems.org/gems/statesman/versions/3.1.0

```
gem 'statesman', '~> 3.4.1'

rails g statesman:active_record_transition Order OrderTransition


```

```ruby
class OrderStateMachine
  include Statesman::Machine
  state :pending, initial: true
  state :checking_out
  state :purchased
  state :shipped
  state :cancelled
  state :failed
  state :refunded
  
  transition from: :pending, to: [:checking_out, :cancelled]
  transition from: :checking_out, to: [:purchaed, :cancelled]
  transition from: :purchased, to: [:shipped, :failed]
  transition from: :shipped, to: :refunded
  
  guard_transition(to: :checking) do |order|
    order.products_in_stock?
  end
  
  before_transition(from: :checking_out, to: :cancelled) do |order, transition|
    order.reallocate_stock
  end
  
  before_transition(to: :purchased) do |order, transition|
    PaymentService.new(order).submit
  end
  
  after_transition(to: purchased) do |order, transition|
    MailerService.order_confirmation(order).deliver
  end
end

class Order < ActiveRecord::Base
  include Statesman::Adapters::ActiveRecordQueries
  
  has_many :order_transitionss, autosave: false
  
  def state_machine
    @state_machine ||= OrderStateMachine.new(self, transition_class: OrderTransiotion)
  end
  
  def self.transition_clas
    OrderTransiton
  end
  
  def self.initial_state
    :pending
  end
  private_class_method :initial_state
end

class OrderTransition < ActiveRecord::Base
  include Statesman::Adapters::ActiveRecordTransition
  
  validates :to_state, inclusion: { in: OrderStateMachine.states }
  
  belongs_to :order, inverse_of: :order_transitions
end

Order.first.state_mechine.current_state
Order.first.state_machine.allowed-transitions
Order.first.state_machine.can_transition_to?(:cancelled)
Order.first.state_machine.transition_to(:cancelled, optional: :metadata)
Order.first.state_machine.transition_to(:cancelled)

Order.in_state(:cancelled)
Order.not_in_state(:checking_out)

# config/initializers/statesman.rb
Statesman.configure do
  storage_adapter(Statesman::Adapters::ActiveRecord)
end



```

```
```

