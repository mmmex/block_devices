# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'open3'
require 'fileutils'

def get_vm_name(id)
  out, err = Open3.capture2e('VBoxManage list vms')
  raise out unless err.exitstatus.zero?

  path = path = File.dirname(__FILE__).split('/').last
  name = out.split(/\n/)
            .select { |x| x.start_with? "\"#{path}_#{id}" }
            .map { |x| x.tr('"', '') }
            .map { |x| x.split(' ')[0].strip }
            .first

  name
end


def controller_exists(name, controller_name)
  return false if name.nil?

  out, err = Open3.capture2e("VBoxManage showvminfo #{name}")
  raise out unless err.exitstatus.zero?

  out.split(/\n/)
     .select { |x| x.start_with? 'Storage Controller Name' }
     .map { |x| x.split(':')[1].strip }
     .any? { |x| x == controller_name }
end


# add NVME disks
def create_nvme_disks(vbox, name)
  unless controller_exists(name, 'NVME Controller')
    vbox.customize ['storagectl', :id,
                    '--name', 'NVME Controller',
                    '--add', 'pcie']
  end

  dir = "../vdisks"
  FileUtils.mkdir_p dir unless File.directory?(dir)

  disks = (0..8).map { |x| ["nvmedisk#{x}_", '1024'] }

  disks.each_with_index do |(name, size), i|
    file_to_disk = "#{dir}/#{name}.vdi"
    port = (i ).to_s

    unless File.exist?(file_to_disk)
      vbox.customize ['createmedium',
                      'disk',
                      '--filename',
                      file_to_disk,
                      '--size',
                      size,
                      '--format',
                      'VDI',
                      '--variant',
                      'standard']
    end

    vbox.customize ['storageattach', :id,
                    '--storagectl', 'NVME Controller',
                    '--port', port,
                    '--type', 'hdd',
                    '--medium', file_to_disk,
                    '--device', '0']

  end
end


def create_disks(vbox, name)
  unless controller_exists(name, 'SATA Controller')
    vbox.customize ['storagectl', :id,
                    '--name', 'SATA Controller',
                    '--add', 'sata']
  end

  dir = "../vdisks"
  FileUtils.mkdir_p dir unless File.directory?(dir)

  disks = (1..10).map { |x| ["disk#{x}_", '1024'] }

  disks.each_with_index do |(name, size), i|
    file_to_disk = "#{dir}/#{name}.vdi"
    port = (i + 1).to_s

    unless File.exist?(file_to_disk)
      vbox.customize ['createmedium',
                      'disk',
                      '--filename',
                      file_to_disk,
                      '--size',
                      size,
                      '--format',
                      'VDI',
                      '--variant',
                      'standard']
    end

    vbox.customize ['storageattach', :id,
                    '--storagectl', 'SATA Controller',
                    '--port', port,
                    '--type', 'hdd',
                    '--medium', file_to_disk,
                    '--device', '0']

    vbox.customize ['setextradata', :id,
                    "VBoxInternal/Devices/ahci/0/Config/Port#{port}/SerialNumber",
                    name.ljust(20, '0')]
  end
end

Vagrant.configure("2") do |config|

config.vm.define "server" do |server|
  #config.vm.box = 'centos/8'
  config.vm.box = 'almalinux/8'
  #config.vm.box_version = "2011.0"
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"
  server.vm.host_name = 'server'
  server.vm.network :private_network, ip: "192.168.56.4"

  server.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "1024"
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

  server.vm.provider 'virtualbox' do |vbx|
      name = get_vm_name('server')
      create_disks(vbx, name)
      create_nvme_disks(vbx, name)
  end

end

end
