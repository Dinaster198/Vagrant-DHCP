Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  # =========================
  # SERVIDOR DHCP
  # =========================

  config.vm.define "srv" do |srv|


    # Adaptador público
    srv.vm.network "public_network", bridge: "enp4s0"

    # Red interna
    srv.vm.network "private_network",
      ip: "192.168.57.10",
      virtualbox__intnet: "intnet"

  end


  # =========================
  # CLIENTE C1
  # =========================

  config.vm.define "c1" do |c1|


    c1.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "intnet"

  end


  # =========================
  # PRINTER
  # =========================

  config.vm.define "printer" do |printer|


    printer.vm.network "private_network",
      type: "dhcp",
      mac: "080027AABBCC",
      virtualbox__intnet: "intnet"

  end

end
