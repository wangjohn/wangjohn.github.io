---
layout: post
title:  "Rails Global Configurations"
date:   2013-07-12 20:31:05
categories: rails gsoc config
---

`activerecord/lib/active_record/railtie.rb`

45: rake_tasks - when setting DatabaseTask
112: active_record.set_configs - when configuring active record configs
122: ...
Anything with: initializer "" do |app|

`activemodel/lib/active_model/railtie.rb`

9: Rails.env.test?, checks the global Rails env

`activesupport/lib/active_support/railtie.rb`

12, 20, 34, 40: Anything with: initializer "" do |app|

`actionview/lib/action_view/railtie.rb`

15, 25, 33: Anything with: initializer "" do |app|

`actionpack/lib/action_controller/railtie.rb`
`actionpack/lib/action_dispatch/railtie.rb`
`actionmailer/lib/action_mailer/railtie.rb`

